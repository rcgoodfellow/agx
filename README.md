# agx
An [AgentX](https://tools.ietf.org/html/rfc2741) library for Go

## Rationale
There are already a few other [AgentX](https://tools.ietf.org/html/rfc2741) libraries for Go out there. However, none of the ones I found seem to support setting variables, and most seem to be built with a relatively static devices in mind. **agx** is purposely designed to support managing highly dynamic devices. Both set and get operations are exposed through functional interfaces that allow your code to be executed when GET or SET operations come through the pipes.

## Disclaimer 
I am still pushing toward an initial release and the library is not yet fully functional.

## Basic Usage
The example below sets up an agent to manage vlans using the [Q-BRIDGE](https://tools.ietf.org/html/rfc4363) standard.
```go
package main

import "github.com/rcgoodfellow/agx"

func main() {
	id, descr := "qbridge-agent", "agent for controlling valns"
	qbridge := "1.3.6.1.2.1.17"
	
	c, err := agx.Connect(&id, &descr)
	defer c.Disconnect()
	
	c.Register(qbridge)
	defer c.Unregister(qbridge)

	c.OnGet(qbridge, func(oid agx.Subtree) agx.VarBind {

		var v agx.VarBind
		v.Type = agx.OctetStringT
		v.Name = oid
		v.Data = *agx.NewOctetString(string([]byte{0xcc, 0x33}))
		return v

	})
	c.OnTestSet(qvs, func(vb agx.VarBind, sessionId int) agx.TestSetResult {

		log.Printf("[test-set] oid::%s session=%d", vb.Name.String(), sessionId)
		
		//do something to test whether the set operation is valid for your device here
		
		return agx.TestSetNoError
	}
	c.OnCommitSet(func(sessionId int) agx.CommitSetResult {

		log.Printf("[commit-set] session=%d", sessionId)
		
		//do something to implement the set here

		return agx.CommitSetNoError

	})

	//wait for connection to close
	<-c.Closed
}
```
