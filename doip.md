# DoIP (UDS)
The definition of ECUs for the Diagnostic communication over Internet Protocol (DoIP) is very similar to ISO-TP as the diagnostic protocol is the same in both cases (UDS). The main difference is the addressing and for the HW simulator there is an additional file to define the server parameters.

## Hardware vs. software simulator
Unlike CAN protocols there is a fundamental difference between Hardware and Software simulators for DoIP.

One instance of the hardware simulator (e.g. CarSimulator) running on one dedicated (virtual) machine (Raspberry Pi, Docker container, ...) can only simulate one single DoIP node. This is because each DoIP node needs its own IP address. The CarSimulator does currently not support running different instances on different IP addresses of the same device.

In contrast a software-only simulation (e.g. AVL DiTEST vega.creator) can simulate multiple virtual DoIP nodes since there is no real network communication, hence no IP addresses involved.
In both cases it is possible to simulate multiple ECUs reachable via the same DoIP Gateway Node.

## Addressing

The header contains the following fields:

* `DoIPLogicalEcuAddress` – Logical Address of the simulated simulated ECU
* `DoIPLogicalGatewayAddress` – Optional Logical Address of the DoIP Gateway Node this ECU is virtually located on or behind.
    * Hardware Simulation: This address is ignored as it is specified globally in the DoIP server configuration.
    * Software Simulation: If this is not present, the ECU will be reachably through any gateway address.

### Example

```lua
PCM = {
    DoIPLogicalEcuAddress = 0x1725,
    DoIPLogicalGatewayAddress = 0x1710,

    Raw = {
        ["10 02"] = "50 02 00 19 01 F4",
        ["22 FA BC"] = "62 FA BC 10 33 11",
    }
}
```

## Configuring the simulated DoIP Node
This is only relevant for the hardware simulator (CarSimulator). Its purpose is to configure the the vehicle announcement / vehicle identification response. Since the software simulator does not simulate vehicle announcement it is not relevant.

You need to create a file calle `doipserver.lua` next to your lua simulations that contains a "Main" section with the following parameters:

* `ANNOUNCE_NUM` – Number of vehicle announcements to send
* `ANNOUNCE_INTERVAL` – Time in milliseconds to wait between vehicle announcements
* `VIN` - VIN to be sent in the vehicle announcement / vehicle identification response
* `LOGICAL_ADDRESS` - Logical Address of this ECU to be sent in the vehicle announcement / vehicle identification response
* `EID` - EID to be sent in the vehicle announcement / vehicle identification response
* `GID` - EID to be sent in the vehicle announcement / vehicle identification response
* `FURTHER_ACTION` - Further Action to be sent in the vehicle announcement / vehicle identification response - currently no further actions are supported so this should be set to 0x00.


### Example
```lua
Main = {
	ANNOUNCE_NUM = 3,
	ANNOUNCE_INTERVAL = 500,			-- in milliseconds

	VIN = "WVWZZZ1JZ3W386752",
	LOGICAL_ADDRESS = 0x1710,
	EID = "000000",
	GID = "000000",
	FURTHER_ACTION = 0x00,
}
```