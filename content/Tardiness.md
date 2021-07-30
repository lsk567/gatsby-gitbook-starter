In certain circumstances in a Lingua Franca program, it is possible that an event occurs at a later tag than what was originally intended. This discrepancy can be interpreted as a `tardiness` with a scalar value equal to the amount by which the original intended logical tag has been missed.  The concept of `tardiness` could either be considered an integral part of the language (where this discrepancy is explicitly allowed) or an error condition that arose due to some violated assumption. Here, we discuss each scenario where `tardiness` can occur.

### Physical Actions

[Physical actions](https://github.com/icyphy/lingua-franca/wiki/Language-Specification#Action-Declaration) are a language feature that are designed to allow a Lingua Franca program to respond to outside events from the physical world. For a physical action `a`, a `schedule` function can (and should) be called outside of a reaction asynchronously with an optional desired delay `d`, and consequently trigger reactions to a particular external event. For a concrete example on how to use physical actions, see [AsyncCallback](https://github.com/icyphy/lingua-franca/blob/master/test/C/AsyncCallback.lf).

For the purposes of this document, let's assume an asynchronous invocation of `schedule` on a physical action `a` with a delay of `d1` has occurred when the Lingua Franca program was at [tag](https://github.com/icyphy/lingua-franca/wiki/Language-Specification#superdense-time) `t1(time, microstep)` and the underlying physical clock was indicating a time `T1`.  In this case, the _intended tag_ of the external event can be interpreted to be `(t1.time + d1,0)`.

There are two scenarios to be considered:

`t1.time + d1 >= T1` : Since `t.time<=T1` when a physical action is present, the intended timestamp at the time of calling `schedule` can be honored. In other words, the Lingua Franca runtime can safely ensure that the corresponding event is in the future and will not interfere with reactions executing at the current tag. In this circumstance, `tardiness` is zero.

`T1 > t1.time + d1`: In this case, the intended logical tag cannot be honored and a new tag in the future must be assigned. For purposes beyond the scope of this document, the future assigned tag in this case will be `(T1, 0)`.  In such a scenario, a tardiness has occurred, where the original intended tag was not honored. The calculated `tardiness` will be `T1 - t1.time - d1`. In code, this tardiness can be accessed via `a->trigger->tardiness`. See [AsyncCallback](https://github.com/icyphy/lingua-franca/blob/master/test/C/AsyncCallback.lf) for a more concrete example of this scenario. Alternatively, the [`Tardy`](#the-tardy-handler) handler could be used as an alternative to the original reaction triggered by the physical action.

**Marten**: I have two comments about this:
1. Tardiness does not make a whole lot of sense when the physical action is scheduled with zero delay. The intended tag in that case is arguably T1. I think tardiness should therefore be zero in that case.
2. I would consider tardiness of a physical action to not be an error condition, so I don't think it should trigger the invocation of an error handler. If the reaction cares about tardiness, it could implement checks in normal reaction code.

**Christian**: I find the definition of 'intended tag' problematic. The physcial action is scheduled asynchronously by an external process. The logical time of the reactor execution has no meaning in this context and its also not under the control of the external process. The logical time just happens to have a certain value when schedule is called.

### Deadlines

In a Lingua Franca program, a [deadline](https://github.com/icyphy/lingua-franca/wiki/Language-Specification#Deadlines) is defined as a bound on the relation between logical time and physical time. More specifically, a deadline `d` "specifies an alternative reaction body `b2` that should be invoked if the *physical* time [`T`] at which the normal reaction `b1` would invoked is greater than the *logical* time [`t`] of the trigger by at least `d`". Inherently, whenever `b2` is triggered, a tardiness has occurred. The amount of this tardiness will be `T - t - d`.

FIXME: How it is accessed and implemented.

### Physical Connections

In a [federated execution](https://github.com/icyphy/lingua-franca/wiki/Distributed-Execution), communication between two federates can either happen on a [physical](https://github.com/icyphy/lingua-franca/wiki/Physical-Connections) (denoted by `~>`) or a logical (denoted by `->`) connection. In the case of physical connections, messages sent over the physical connection will be assigned a timestamp `t` at the sender with an optional delay `d` imposed using `after`. In this scenario, the _intended tag_ of the message is thus considered to be `t_intended = (t.time + d,t.microstep)`. At the receiving end, the tag of the remote federate will be `t'`, where it is possible that `t != t'`. Similar to physical actions, there will be two scenarios:

`t_intended > t'`: In this scenario, the original intended tag is in the future and thus can be honored.

`t_intended <= t'`: In this scenario, the original intended tag cannot be feasibly honored on the receiver without breaking the causality between past events (or events currently in process) and future events. In this case, a new tag must be assigned to the message. For consistency, physical connections will follow physical actions, and assign `t_new = (T',0)` (where T is the physical time at the receiver and `t'.time < T'`). In this case, a tardiness has occurred and the calculated value would be `(T1 - t.time - d,0)`. In code, this tardiness can be accessed on the input port of the reaction at the receiver by accessing `port->tardiness`. See [DistributedCountDecentralized.lf]() for a more concrete example of this scenario. Alternatively, the [`Tardy`](#the-tardy-handler) handler could be used as an alternative to the original reaction.

### Logical Connections

As was mentioned in the previous section, federates can use a logical connection (`->`) to send message to each other over the network. By definition, logical connections differ from physical connections in that they will make a concerted effort to keep and apply the original _intended tag_ of the message. 

However, the network channel is inherently unreliable. To remedy this limitation, by default, messages exchanged between the federates will go through the **runtime infrastructure (RTI)** and each federate will [ask](https://github.com/icyphy/lingua-franca/wiki/Distributed-Execution#communication-between-federates) the RTI before advancing its logical time. This mechanism ensures that tardiness will never occur. In code, this mode can be explicitly enabled by setting the `coordination` target property to `centralized` (although this is the default). For a more concrete example, see [DistributedCount.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCount.lf). 

However, the federated synchronized barrier on logical time might add extra overhead for certain applications. This barrier on logical time can be eliminated by using the `decentralized` coordination. To clarify this mode of coordination, imagine a scenario where the sender with current logical tag `t` assigns a tag `t_intended = (t.time + d, t.microstep)` to an outgoing message. At the receiving end with a logical tag `t'`, similar to the case for physical connections, two scenarios can arise:

`t_intended > t'`: In this scenario, the original intended tag is in the future and thus can be honored. Specifically, either the delay `d` (specified using an `after`) is sufficiently large to cover the inherent latencies (and variations) in such a system or the receiver, via an alternative mechanism, has been able to hold its logical time `t'` long enough to receive this message. This alternative mechanism is beyond the scope of this document. For more detailed description on how this can be achieved and the inherent latencies involved, see [[Ptides]].

`t_intended <= t'`: In this scenario, the spirit of logical connections has been violated since the intended tag of the message cannot be honored. Instead, an alternative tag must be assigned to the incoming message. Because logical connections are considered a mission-critical component, this tag will be the smallest possible tag, namely, `t'_new = (t'.time, t'.microstep + 1)`. In this case, a tardiness has occurred with a `tardiness` amount equal to `t'_new - t_intended`. In code, this tardiness can be accessed on the input port of the reaction at the receiver by accessing `port->tardiness`. See [DistributedCountDecentralized.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountDecentralized.lf) for a more concrete example. Alternatively, the [`Tardy`](#the-tardy-handler) handler could be used as an alternative to the original reaction.

### The Tardy Handler

To facilitate handling of tardiness in a unified manner, a `Tardy` handler has been added to the language, where it can be used as an alternative reaction body to a reaction:

```c
reaction(in) {=
    // Code that can assume
    // no tardiness has occured.
=} tardy {=
    // Code that will deal
    // with tardiness.
=}
```

What is unique to `tardy` is that it is an optional modifier even if `tardiness` is present in the system. If a `Tardy` handler is added to a reaction and a `tardiness` value is present, the `Tardy` handler will be invoked instead of the original reaction.

In other words, the presence of the `tardy` handler can facilitate an assumption that the body of the original reaction can be executed in a such a way that assumes a tardiness will never occur. If a `tardiness` value is present and it is not handled in a `tardy` handler, the `tardiness` value will be propagated downstream because the aforementioned assumption is also violated for every downstream reaction triggered by the original reaction.



For more examples of using the `tardy` handler on the aforementioned scenarios, see:

**Physical Connections**

[DistributedCountPhysicalLate.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountPhysicalLate.lf)

[DistributedCountPhysicalDecentralized.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountPhysicalDecentralized.lf)

[DistributedCountPhysicalDecentralizedLate.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountPhysicalDecentralizedLate.lf)

**Logical Connections**

[DistributedCountDecentralizedLate.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountDecentralizedLate.lf)

[DistributedCountDecentralizedLateDownstream.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountDecentralizedLateDownstream.lf)

[DistributedCountDecentralizedLateHierarchy.lf](https://github.com/icyphy/lingua-franca/blob/master/test/C/DistributedCountDecentralizedLateHierarchy.lf)