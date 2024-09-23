---
title: grpc-负载均衡
date: 2024-09-23 15:54:17
categories:
- 中间件源码
- grpc
tags:
---

# grpc-负载均衡

这里我们首先看一下dnsResolver内置的是如何实现的负载均衡
```golang
type dnsResolver struct {
	host     string
	port     string
	resolver internal.NetResolver
	ctx      context.Context
	cancel   context.CancelFunc
	cc       resolver.ClientConn
	// rn channel is used by ResolveNow() to force an immediate resolution of the
	// target.
	rn chan struct{}
	// wg is used to enforce Close() to return after the watcher() goroutine has
	// finished. Otherwise, data race will be possible. [Race Example] in
	// dns_resolver_test we replace the real lookup functions with mocked ones to
	// facilitate testing. If Close() doesn't wait for watcher() goroutine
	// finishes, race detector sometimes will warn lookup (READ the lookup
	// function pointers) inside watcher() goroutine has data race with
	// replaceNetFunc (WRITE the lookup function pointers).
	wg                   sync.WaitGroup
	disableServiceConfig bool
}
```
watch机制
```golang
func (d *dnsResolver) watcher() {
	defer d.wg.Done()
	backoffIndex := 1
	for {
		state, err := d.lookup()
		if err != nil {
			// Report error to the underlying grpc.ClientConn.
			d.cc.ReportError(err)
		} else {
			err = d.cc.UpdateState(*state)
		}

		var nextResolutionTime time.Time
		if err == nil {
			// Success resolving, wait for the next ResolveNow. However, also wait 30
			// seconds at the very least to prevent constantly re-resolving.
			backoffIndex = 1
			nextResolutionTime = internal.TimeNowFunc().Add(MinResolutionInterval)
			select {
			case <-d.ctx.Done():
				return
			case <-d.rn:
			}
		} else {
			// Poll on an error found in DNS Resolver or an error received from
			// ClientConn.
			// 这里默认是30s重新执行
			nextResolutionTime = internal.TimeNowFunc().Add(backoff.DefaultExponential.Backoff(backoffIndex))
			backoffIndex++
		}
		select {
		case <-d.ctx.Done():
			return
		case <-internal.TimeAfterFunc(internal.TimeUntilFunc(nextResolutionTime)):
		}
	}
}

//然后lookup方法得到ip的list
func (d *dnsResolver) lookup() (*resolver.State, error) {
	ctx, cancel := context.WithTimeout(d.ctx, ResolvingTimeout)
	defer cancel()
	srv, srvErr := d.lookupSRV(ctx)
	addrs, hostErr := d.lookupHost(ctx)
	if hostErr != nil && (srvErr != nil || len(srv) == 0) {
		return nil, hostErr
	}

	state := resolver.State{Addresses: addrs}
	if len(srv) > 0 {
		state = grpclbstate.Set(state, &grpclbstate.State{BalancerAddresses: srv})
	}
	if !d.disableServiceConfig {
		state.ServiceConfig = d.lookupTXT(ctx)
	}
	return &state, nil
}
```

所有的resovler都需要对balancer进行初始化，默认是grpclb策略，
```golang
func newCCBalancerWrapper(cc *ClientConn) *ccBalancerWrapper {
	ctx, cancel := context.WithCancel(cc.ctx)
	ccb := &ccBalancerWrapper{
		cc: cc,
		opts: balancer.BuildOptions{
			DialCreds:       cc.dopts.copts.TransportCredentials,
			CredsBundle:     cc.dopts.copts.CredsBundle,
			Dialer:          cc.dopts.copts.Dialer,
			Authority:       cc.authority,
			CustomUserAgent: cc.dopts.copts.UserAgent,
			ChannelzParent:  cc.channelz,
			Target:          cc.parsedTarget,
			MetricsRecorder: cc.metricsRecorderList,
		},
		serializer:       grpcsync.NewCallbackSerializer(ctx),
		serializerCancel: cancel,
	}
	ccb.balancer = gracefulswitch.NewBalancer(ccb, ccb.opts)
	return ccb
}

//具体调用的pick选取连接
func (p *lbPicker) Pick(balancer.PickInfo) (balancer.PickResult, error) {
	p.mu.Lock()
	defer p.mu.Unlock()

	// Layer one roundrobin on serverList.
	s := p.serverList[p.serverListNext]
	p.serverListNext = (p.serverListNext + 1) % len(p.serverList)

	// If it's a drop, return an error and fail the RPC.
	if s.Drop {
		p.stats.drop(s.LoadBalanceToken)
		return balancer.PickResult{}, status.Errorf(codes.Unavailable, "request dropped by grpclb")
	}

	// If not a drop but there's no ready subConns.
	if len(p.subConns) <= 0 {
		return balancer.PickResult{}, balancer.ErrNoSubConnAvailable
	}

	// Return the next ready subConn in the list, also collect rpc stats.
	sc := p.subConns[p.subConnsNext]
	p.subConnsNext = (p.subConnsNext + 1) % len(p.subConns)
	done := func(info balancer.DoneInfo) {
		if !info.BytesSent {
			p.stats.failedToSend()
		} else if info.BytesReceived {
			p.stats.knownReceived()
		}
	}
	return balancer.PickResult{SubConn: sc, Done: done}, nil
}
```

在我们使用的这一版中 没有了重连
```golang
if idle {
        ac.connect()
    }
```

这里实现我们自己的负载均衡 我们有两种方案：（轮询是grpc默认的）
- 加权：这里应该grpc不支持把权重直接加入address中，可以把权重加入address的元数据中，设置新的attributes
- 最小连接
重写一个picker