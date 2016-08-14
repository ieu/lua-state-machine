Lua Finite State Machine
========================================

This is a finite state machine lua micro framework ported from [Javascript Finite State Machine](https://github.com/jakesgordon/javascript-state-machine).

Please note that **NOT** all function that Javascript Finite State Machine provides is implemented.

Usage
=====

Include `state-machine.lua`.

In its simplest form, create a standalone state machine using:

    fsm = StateMachine.create({
      initial = 'green',
      events = {
        { name = 'warn',  from = 'green',  to = 'yellow' },
        { name = 'panic', from = 'yellow', to = 'red'    },
        { name = 'calm',  from = 'red',    to = 'yellow' },
        { name = 'clear', from = 'yellow', to = 'green'  }
      }
    });

... will create an object with a method for each event:

 * fsm:warn()  - transition from 'green' to 'yellow'
 * fsm:panic() - transition from 'yellow' to 'red'
 * fsm:calm()  - transition from 'red' to 'yellow'
 * fsm:clear() - transition from 'yellow' to 'green'

along with the following members:

 * fsm.current       - contains the current state
 * fsm:is(s)         - return true if state `s` is the current state
 * fsm:can(e)        - return true if event `e` can be fired in the current state
 * fsm:cannot(e)     - return true if event `e` cannot be fired in the current state

Multiple 'from' and 'to' states for a single event
==================================================

If an event is allowed **from** multiple states, and always transitions to the same
state, then simply provide an array of states in the `from` attribute of an event. However,
if an event is allowed from multiple states, but should transition **to** a different
state depending on the current state, then provide multiple event entries with
the same name:

    fsm = StateMachine.create({
      initial = 'hungry',
      events = {
        { name = 'eat',  from = 'hungry',                                to = 'satisfied' },
        { name = 'eat',  from = 'satisfied',                             to = 'full'      },
        { name = 'eat',  from = 'full',                                  to = 'sick'      },
        { name = 'rest', from = {'hungry', 'satisfied', 'full', 'sick'}, to = 'hungry'    },
      }
    });

This example will create an object with 2 event methods:

 * fsm:eat()
 * fsm:rest()

The `rest` event will always transition to the `hungry` state, while the `eat` event
will transition to a state that is dependent on the current state.

>> NOTE: The `rest` event could use a wildcard '*' for the 'from' state if it should be
allowed from any current state.

>> NOTE: The `rest` event in the above example can also be specified as multiple events with
the same name if you prefer the verbose approach.

Callbacks
=========

4 types of callback are available by attaching methods to your StateMachine using the following naming conventions:

 * `onbeforeEVENT` - fired before the event
 * `onleaveSTATE`  - fired when leaving the old state
 * `onenterSTATE`  - fired when entering the new state
 * `onafterEVENT`  - fired after the event

>> (using your **specific** EVENT and STATE names)

In addition, 4 general-purpose callbacks can be used to capture **all** event and state changes:

 * `onbeforeevent` - fired before *any* event
 * `onleavestate`  - fired when leaving *any* state
 * `onenterstate`  - fired when entering *any* state
 * `onafterevent`  - fired after *any* event

All callbacks will be passed the same arguments:

 * **event** name
 * **from** state
 * **to** state
 * _(followed by any arguments you passed into the original event method)_

Callbacks can be specified when the state machine is first created:

    fsm = StateMachine.create({
      initial = 'green',
      events = {
        { name = 'warn',  from = 'green',  to = 'yellow' },
        { name = 'panic', from = 'yellow', to = 'red'    },
        { name = 'calm',  from = 'red',    to = 'yellow' },
        { name = 'clear', from = 'yellow', to = 'green'  }
      },
      callbacks = {
        onpanic =  function(event, from, to, msg) print('panic! ' .. msg)    end,
        onclear =  function(event, from, to, msg) print('thanks to ' .. msg) end,
        ongreen =  function(event, from, to)      print('green')             end,
        onyellow = function(event, from, to)      print('yellow')            end,
        onred =    function(event, from, to)      print('red')               end,
      }
    })

    fsm:panic('killer bees')
    fsm:clear('sedatives in the honey pots')
    ...

Additionally, they can be added and removed from the state machine at any time:

    fsm.ongreen      = null
    fsm.onyellow     = null
    fsm.onred        = null
    fsm.onenterstate = function(event, from, to) print(to) end;


The order in which callbacks occur is as follows:

>> assume event **go** transitions from **red** state to **green**

 * `onbeforego`    - specific handler for the **go** event only
 * `onbeforeevent` - generic  handler for all events
 * `onleavered`    - specific handler for the **red** state only
 * `onleavestate`  - generic  handler for all states
 * `onentergreen`  - specific handler for the **green** state only
 * `onenterstate`  - generic  handler for all states
 * `onaftergo`     - specific handler for the **go** event only
 * `onafterevent`  - generic  handler for all events

You can affect the event in 1 way:

 * return `ASYNC` from an `onleaveSTATE` handler to perform an asynchronous state transition
