# ISO-TP protocols (e.g. UDS, KWP)
The first protocol supported by the Lua Simulation Format is ISO-TP.

The header contains the following fields:

* `CanPhysReqId` – CAN ID for frames directed to the simulated ECU
* `CanRespUSDTId` - CAN ID for frames directed from the simulated ECU to the tester
* `CanFuncReqId` - CAN ID for functional request frames
* `CanPhysReqExtAddr` - (optional) Extended address for frames directed to the simulated ECU
* `CanRespUSDTExtAddr` - (optional) Extended address for frames directed from the simulated ECU to the tester

Currently 11bit and 29bit CAN-IDs can be specified.

Request/Response pairs are defined in a section called "Raw".

## Example

```lua
PCM = {
    CanPhysReqId = 0x100, -- or RequestId = 0x100,
    CanRespUSDTId = 0x200, -- or ResponseId = 0x200,
    CanFuncReqId = 0x300, -- or RequestFunctionalId = 0x300,
    CanPhysReqExtAddr = 0x10, -- optional
    CanRespUSDTExtAddr = 0xf1, -- optional
    
    Raw = {
        ["10 02"] = "50 02 00 19 01 F4",
        ["22 FA BC"] = "62 FA BC 10 33 11",
    }
}
```

## Sending multiple responses
In case the simulated ECU would send multiple responses for one specific request, the function `link.send` can be used.

Example:

```lua
PCM = {
    CanPhysReqId = 0x100,
    CanRespUSDTId = 0x200,
    CanFuncReqId = 0x300,

    Raw = {
        ["10 02"] = "50 02 00 19 01 F4",
        ["22 FA BC"] = function(request, link)
            link.send("62 FA BC 10 33 11")
            link.send("62 FA BC 10 33 12")
            link.send("62 FA BC 10 33 13")
        end
    }
}
```
