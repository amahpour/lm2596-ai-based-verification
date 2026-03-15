| PIN.NO. | PIN.NAME | I/O.I/O | DESCRIPTION.DESCRIPTION |
| --- | --- | --- | --- |
| 1 | V IN | I | This is the positive input supply for the IC switching regulator. A suitable input bypass capacitor must be present at this pin to minimize voltage transients and to supply the switching currents required by the regulator. |
| 2 | Output | O | Internal switch. The voltage at this pin switches between approximately (+V IN - V SAT ) and approximately -0.5 V, with a duty cycle of V OUT / V IN . To minimize coupling to sensitive circuitry, the PCB copper area connected to this pin must be kept to a minimum. |
| 3 | Ground | - | Circuit ground |
| 4 | Feedback | I | Senses the regulated output voltage to complete the feedback loop. |
| 5 | ON/OFF | I | Allows the switching regulator circuit to be shut down using logic signals thus dropping the total input supply current to approximately 80 µA. Pulling this pin below a threshold voltage of approximately 1.3 V turns the regulator on, and pulling this pin above 1.3 V (up to a maximum of 25 V) shuts the regulator down. If this shutdown feature is not required, the ON/OFF pin can be wired to the ground pin or it can be left open. In either case, the regulator will be in the ON condition. |
