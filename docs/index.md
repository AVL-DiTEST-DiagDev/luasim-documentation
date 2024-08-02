# Lua Simulation Format
The Lua Simulation Format is a simple, yet powerfull description language to simulate the diagnostic communication capabilities of Electronic Control Units (ECUs).
While this format has a strong focus on diagnostics it can be easily extended to simulate other communication as well. Futher development might bring e.g. restbus simulation comprised of cyclic CAN messages.

Lua is a lightweight scripting language optimized to be used in embedded systems and created to be integrated in software applications for the purpose of easy scriptability.

The Lua Simulation Format is simply a Lua script written in a way such that a simulation engine can understand and run the simulation. In particular that means the Lua file describes all the messages to be simulated while the simulation engine does all the hard work of receiving and sending messages from/to the bus.

With the Lua language underneath it is easy to simulate both, simple communication as well as complex dynamic behavior of ECUs. The Lua Simulation Format allows to define static messages but also use the full functionality of the Lua programming language wherever necessary.

Find more information about Lua at https://www.lua.org/

## Supported Protocols
This page defines the general syntax of the Lua Simulation Format with some protocol specific examples.

Please find more about how to define protocol specific properties in the Lua simulation format here:

* [ISO-TP protocols (e.g. UDS, KWP)](isotp.md)
* [J1939](j1939.md)


## Simulation structure
All simulation files need to have the extension **.lua** to make sure they are detected by the simulation engine.

* A single file can describe one or more ECUs
* One vehicle simulation can consist of one or multiple simulation files, the engine loads and interprets each one independenly.

A simulation description of an individual ECU is organized as a lua table. It consists of

* a header containing meta information such as ECU addresses
* One or more protocol specific sub-tabes containing the actual simulation data.

The following example shows a basic UDS simulation:

```lua
PCM = { -- ECU name
    -- header
    RequestId = 0x100,
    ResponseId = 0x200,
    RequestFunctionalId = 0x300,

    -- UDS Request/Response pairs
    Raw = {
        ["10 02"] = "50 02 00 19 01 F4",
        ["22 FA BC"] = "62 FA BC 10 33 11",
    }
}
```

If you want to add a second ECU to the file, just start a new table underneath.
```lua
PCM = {
    RequestId = 0x100,
    ResponseId = 0x200,
    RequestFunctionalId = 0x300,

    Raw = {
        ["10 02"] = "50 02 00 19 01 F4",
        ["22 FA BC"] = "62 FA BC 10 33 11",
    }
}

BCM = {
    RequestId = 0x110,
    ResponseId = 0x210,
    RequestFunctionalId = 0x300,

    Raw = {
        ["10 02"] = "50 02 00 17 22 34",
        ["22 FA BC"] = "62 FA BC 10 33 12",
    }
}
```

**Important:** Don't forget the *comma (,)* at the end of each line in the table.

## Going dynamic
When it comes to more complex simulations, you can inline Lua program code into your responses. There are a few ways to do that, let's start with a simple one:

```lua
    Raw = {
        ["22 FA BC"] = "62 FA BC " .. ascii("My Response"),
    }
}
```
This converts the Ascii Test "My Response" into a hex string and appends it (using the .. notation) to the first three response bytes.

In fact this results in the same as the following.
```lua
    Raw = {
        ["22 FA BC"] = "62 FA BC 4d 79 20 52 65 73 70 6f 6e 73 65",
    }
```

**Note:** At the moment, function calls inlined into the response are statically only evaluated when the simulator starts

To use the full potential of Lua, you can also render responses using lua programming code.

```lua
    Raw = {
        ["22 FA BC"] = function(request)
            myResponse = "62 FA BC"
            myResponse = MyResponse .. ascii("My Response")
            return myResponse
        end,
    }
```

## Simulation helper functions

In order to make writing simulations easier there are a few helper functions integrated. One of them you have already seen but there are more.

You can use the following function calls inside your dynamic Lua code

* `ascii(string)` – Converts a UTF-8 string into a lexical hexadecimal byte string
* `sleep(number)` – Sleeps the amount in milliseconds before proceeding any further

Furthermore there are some Lua simulation library functions already included and usable.
Please find code as well as documentation here: https://github.com/AVL-DiTEST-DiagDev/lua-simulation-libraries

Example:
```lua
    Raw = {
        ["22 F1 91"] = "62 F1 91" .. ascii("SALGA2EV9HA298784"),

        ["19 02 AF"] = function (request)
            sendRaw("7F 19 78")           -- send Response pending immediatly
            sleep(5000)                   -- wait 5 seconds
            return "59 02 FF E3 00 54 2F" -- send the final response
        end
    },
```

## Creating custom Lua libraries
The simulator contains a basic Lua language interpreter with a few simulator specific functions added. If you write complex simulations and want to reuse code, you should consider putting that code into separate functions and saving them into a separate Lua file.

For example, create a file called **lualib.lua** next to your simulation file and add the following content:

```lua
function toHex(intValue, numBytes)
  return string.format('%0' .. numBytes*2 .. 'X', intValue)
end
```

Now include that file into your simulation by adding at the very beginning:
```lua
dofile(lualib.lua)
```

You will then be able to use the function in that file in your simulation.

```lua
    Raw = {
        ["22 FA BC"] = "62 FA BC " .. toHex(17, 1)
    }
```

**Note:** The path to the included lua file is always relative to the first lua file that contains the simulation. That means, if you include a library that is located in a different directory and then include another library from that one, the path is still relative to the original lua file.

## Global variables
When you need to share state between function calls of requests, you can define global variables in the lua file.

```lua
-- define global variables outside the ECU table
ReadDTCResponse = "C0 01 81 28"
EngineRPM = 1000

-- ECU table starts here
PCM = {
   RequestId = 0x7E0,
   ResponseId = 0x7E8,
   RequestFunctionalId = 0x7DF,

   Raw = {
      ["3E 00"] = "7E",
      ["19 02 AF"] = function () 
			return "59 02 FA" .. ReadDTCResponse
    	end,
	  ["14 FF FF FF"] = function ()
			ReadDTCResponse = "";
			return "54";
		end,
	  ["22 D0 19"] = function ()
		EngineRPM = EngineRPM + 10
		if EngineRPM > 2000 then
			EngineRPM = 1000
		end
				
		return "62 D0 19" .. toByteResponse(EngineRPM,2)
	  end,
   }
}
```

## A few notes on Lua syntax
* Comments start with two dashes (--). Everything behind that on the same line will be ignored.
* Simulations are organized in lua tables. All entries inside the tables need to be separated by comma (,). It is fine to have a comma after the last entry in the table as well.
* Ensure you don't forget the comma after the "end" of an inline function
* Statements inside functions do not need a separating character
* The blocks of control structures (branches, loops) and functions finish with "end".
  As opposed to other languages the block is not enclosed in curly braces nor does the indentation matter
* You can concatenate strings using two dots (..)

You will find more about Lua and the syntax at https://www.lua.org/manual/5.1/
