## sw/573/SAM1tester/doc/manual.md &nbsp;&nbsp;&nbsp;&nbsp; 5-12-2020 
___
Commands for the SAM1 tester are text scripts.  
Command execution leads to result text, following the rules:
1. Commands are separated with LF (ASCII-code 10).
2. Command evaluation takes place in two steps:
   1. break the command down into words,
   1. use the first word to locate the command routine,  
      call the routine using the other words as parameters to the routine.
3. Words of commands are separated by whitespace (32, 9-13) except LF.
10. Commands with # as the first character are treated as comments.
12. see rule (12) of [rfc1925.html](https://tools.ietf.org/html/rfc1925)

SAM1 tester can be controlled by a serial terminal through an FTDI TTL-232R cable.  
It can be either 5V or 3V3 type, and the terminal settings are `115200,n,8,2`.  
Commands can be interactively typed in, with a command prompt `% ` and just two  
special keyboard keys for editing:
1. __Enter__ to enter the command
2. __Backspace__ to remove the previous character

The command `info commands` shows a list of the available commands.  
Commands can be abbreviated as long as they don't become ambiguous, so entering `i c`  
at the time of writing this manual would resolve to the same list of the available commands  
as would typing the complete words. With the two commands `enable` and `error` one  
has to enter at least the first two tokens `en` or `er` to choose between them.

The available commands at the time of writing this manual, with links to the description  
(to be) implemented are:

name   |function
:---|:---
[?](#help)         |display the above rules
[command](#command)|forward a command to the device under test
[enable](#enable)  |set and/or read enable bits
[error](#error)    |clear error bits
[info](#info)      |show available subcommands
[pin](#pin)        |set and/or read microcontroller pins
[test](#test)      |test microcontroller peripherals
[var](#var)        |set and/or read variable values

The command descriptions below have question marks around optional parameters and  
ellipsis ... to indicate the possibility of repeating a part before them. The term `vbe` in  
the descriptions is a special pattern consisting of at most three values with a point __.__ as  
value separator and can look (without question marks) as:

pattern|description
:---|:---
*value*|just a value
*value*__.__?*begin*?|value and optional begin value
*value*__.__?*begin*??__.__*end*?|value and optional begin and end values
__.__?*begin*??__.__*end*?|just optional begin and end values

The `vbe` is being used when value, begin and end have default values if not filled in.  
The implemented commands can be described as:

<a name="help"></a>
### ?
__?__  
Show the rules of SAM1 tester (until now in the Dutch language).  
The purpose of this command is showing that command names can contain non-alphanumerical tokens.  
The command results in text like the start of this manual, with "SAM1 tester" replaced by "microcontroller".  




<a name="command"></a>
### command
__command__ ?*arg ...*?  
Let SAM1 tester send a command to the device under test (DUT).  
All arguments after __command__ are forwarded to the DUT.  
This command has no result, but lines received from the DUT will be returned through  
the FTDI-interface preceded by `<` and a space, so the driver program of SAM1 tester  
knows the difference between an answer from the tester and the DUT.  
*examples*:  
`command info commands` ask for all known commands of the DUT.  
`command var -1.` ask for the identification of the DUT.  
`command pin A3 o1` ask the DUT to set its port A pin 3 as output high.  
`command command var -1.` ask for the identification of the device connected to the DUT.  


<a name="enable"></a>
### enable
__enable__ ?*pattern*? ?*value ...*?  
Disable/enable one or more parts of SAM1 tester functionality.  
The *pattern* is a `vbe`-word restricting the range of the command:  
1. ?*value*? -- *value* is treated like an optional ?*value ...*?  
2. ?*value*?.?*begin*? --  the same, now starting at index *begin*.  
3. ?*value*?.?*begin*?.?*end*?. -- the same, now ending no further than index *end*.  

*Value* can be one of `- 0 1`, where `-` will skip one index and the other values  
will store the value at that index, as long as it's within the restricted index range.  
The range restriction *begin* defaults to 0 and *end* defaults to 15.  
The result of this command is a readout list of the range determined by *pattern*.  
*examples*:  
`enable` show enable bits from index 0  
`enable 1` set the enable bit at index 0 to 1, show enable bits from index 0  
`enable 1.5` set the enable bit at index 5 to 1, show enable bits from index 5  
`enable .5 - 1` skip the enable bit at index 5, set the next bit to 1, show bits from index 5  
`enable -.5.8 1` skip the enable bit at index 5, set the bit at index 6 to 1, show bits 5 to 8  


<a name="error"></a>
### error
__error__ *pattern* ?*value ...*?  
Clear one or more errors that have occurred when using SAM1 tester functionality.  
The *pattern* is a `vbe`-word with the same functionality as in the __enable__ command.  
*Value* can be one of `- 0 1`, where `1` will reset an error bit and the other values do nothing.  
The range restriction *begin* defaults to 0 and *end* defaults to 31.  
The result of this command is a readout list of the range determined by *pattern*.  
*examples*:  
`error` show error bits from index 0  
`error 1` clear the error bit at index 0, show error bits from index 0  
`error -.5.8 1` skip the error bit at index 5, clear the bit at index 6, show bits 5 to 8  


<a name="info"></a>
### info
__info__ *subcommand*  
At the moment only the subcommands `commands` and `tests` are implemented, and choosing one  
will return a list with the known commands or tests.  
*examples:*  
`info commands` show all known commands  
`info tests` show all known tests  


<a name="pin"></a>
### pin
__pin__ ?*pattern*? ?*value ...*?  
SAM1 tester IO-pin functionality can be changed to accomodate tests. *Pattern* can be:
- empty -- to show direction and level of all available port pins
- one token -- to restrict the command to a specific port
- two tokens -- to restrict the command to a specific port and start with a specific pin

If the command has *value* parameter(s) after *pattern* they will be used to set IO-pin functionality.  
*Value* can be any one token from `d u i n o p - 0 1` or one from `n o p` followed by one of `0 1`.  
The resulting port pin(s) after the command will be:  
`-` pin not touched by this command  
`d` input with Rpd enabled (not available on RX13T)  
`u` input with Rpu enabled  
`i` input without Rpd/Rpu enabled  
`n` output with N-type open-drain  
`o` ordinary output, push-pull type  
`p` output with P-type open-drain (not available on RX13T)  
`0` low if the pin was an output or 0 is input after one from `n o p`  
`1` high if the pin was an output or 1 is input after one from `n o p`  
The result of this command shows one from `L H` for input pins, one from `0 1` for output pins,  
and `-` for pins that are not available. Ports not available on the microcontroller, like port 6,  
will show no output line with the empty __pin__ call.  
*examples:*  
`pin` show the values of all available port pins  
`pin A` show only the values of port A pins  
`pin A3 o1` set pin A3 to output with bit value '1'  
__NB__: for each line the first value is the port name, followed by pin values in Little-endian order.  
__NNB__: No peripheral function is changed with this command. The command test is meant for that.  


<a name="test"></a>
### test
__test__ *subcommand* ?*arg ...*?  
To be implemented.


<a name="var"></a>
### var
__var__ *pattern* ?*value ...*?  
SAM1 tester variables consist of settings and other values that each can be accessed through  
a numbered identifier called an `infonr`. An infonr for settings is a negative number and an  
infonr for other values is a non-negative number.  
As an infonr is likely to change with a software update, the infonrs are gathered in groups  
that can be identified with a group number `groupnr`. A groupnr for settings is a negative  
number and a groupnr for other variables is non-negative.  
As groupnr -1 is used for identifying the device and software its values get text formatted  
before being output and reset to their default values after a device reset.  
The *pattern* is a `vbe`-word restricting the range of the command:  
1. *value* -- will restrict the command to the group containing the `infonr`  *value*.
2. `-` -- will show the values of all infonr with negative index, one line per group.
3. `+` -- will show the values of all infonr with non-negative index, one line per group.
4. __.__?*begin*??__.__*end*? -- will restrict *begin* as well as *end* to a valid `infonr`.
5. *group*__.__?*begin*??__.__*end*? -- will restrict *group* to a valid `groupnr` and treat  
*begin* and *end* as indices that will be restricted to within the group.

If the command has *value* parameters after *pattern* they will be stored within the range.  
In short, if *pattern* contains a point, but not at the beginning, the given `groupnr`, otherwise  
the first given `infonr` determines how to restrict the command range.  
The result of this command is a readout list of the range determined by *pattern*.  
*examples:*  
`var - 3` show the values of all infonr with negative index, parameter 3 is discarded.  
`var -3` show the values of all infonrs in the group containing `infonr` -3.  
`var .-3` show the values of all infonrs in the group containing `infonr` -3.  
`var .-3.-9` show the values of `infonr` -3 to -9, even if not in the same group.  
`var 3 3 4 5 6 7 8 9 10` set values 3 to 10 at `infonr` 3 to 10, restricted to the starting group.  
`var .3.10 3 4 5 6 7 8 9 10` set values 3 to 10 at `infonr` 3 to 10, even if not in the same group.  
`var .3.10 3 4 - 6 7 8 - 10` almost the same, now leaving `infonr` 5 and 9 untouched.  

`var 2.` show the values of all infonrs in `groupnr` 2.  
`var 2.2` show the values of infonrs in `groupnr` 2 from index 2 to the end of the group.  
`var 2..2` show the values of infonrs in `groupnr` 2 from index 0 to index 2.  
`var 2.0.1 2 3 4` set values 2 to 4 in groupnr, restricted within indices 0 and 1.  
`var 2.0 1` set the value of the first `infonr` in `groupnr` 2 to 1.  
`var 2. 2` set the value of the first `infonr` in `groupnr` 2 to 2.  
`var 2.. 3` set the value of the first `infonr` in `groupnr` 2 to 3.  
__NB__: for each line the value between parentheses is the infonr of the first variable.  
