## sw/573/SAM1tester/doc/manual.htm ` ` 4-12-2020 
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
[var](#var)         |set and/or read variable values

The command descriptions below have question marks around optional parameters and  
ellipsis ... to indicate the possibility of repeating a part before them. The term `vbe` in  
the descriptions is a special word consisting of at most three values with a point __.__ as  
value separator and can look (without question marks) as:

word form|description
:---|:---
*value*|just a numerical value
*value*__.__*begin*|value and begin value
*value*__.__*begin*__.__*end*|value and begin value and end value  

The `vbe` is being used when value, begin and end have default values if not filled in.  
The implemented commands can be described as:

<a name="help"></a>
#### ?

<a name="command"></a>
#### command

<a name="enable"></a>
### enable
__enable__ *vbe* ?*value ...*?  
Used for disabling/enabling parts of SAM1 tester functionality. Without further arguments  
it lists the enable bit values, and with only one value changes the enable bit at index 0.  
The *vbe*-value *begin* defaults to 0, *end* defaults to 15.  
Value can be any from `- 0 1`, where a - means don't change the value.  
`enable 1.8` sets enable bit with index 8 to the value '1' and shows indices 8 to end.  
`enable 1.8.10` sets enable bit with index 8 to the value '1' and shows bits with index 8 to 10.  
*enable .8* -- shows enable bits from index 8 upwards and might be used for easier  
 counting of which bit one wants to influence.


<a name="error"></a>
#### enable


<a name="info"></a>
### info
__info__ *subcommand*  
At the moment only the subcommands `commands` and `tests` are implemented, and choosing one  
will return a list with the possible commands or tests.

<a name="pin"></a>
### pin
__pin__ ?*pattern*? ?value ...?  
The functionality of SAM1 tester IO-pins can be changed with this command. Pattern can be  
empty to show all available port pins, or the port name with optional bit number to start with  
when entering new values. Values read back  show L or H for inputs, 0 or 1 four outputs, and - for  
pins that are not available on the microcontroller port. Ports not available on the microcontroller,  
like port 6, will show no output line.  
Value can be any from `d u i n o p 0 1`, with the first three for setting a pin to input while  
enabling Rpd or Rpu, or disabling these (Rpd not available in RX13T), and the other letters for  
setting a pin to output while also changing its open-drain type to N, none or P respectively  
(open-drain P-type not available in RX13T).  
The values 0 and 1 can be added after any from `n o p` to also set the level, or without the letter  
to only set the level if the pin is an output pin.  
*examples:*  
`pin` show the values of all available port pins  
`pin A` show only the values of port A pins  
`pin A3 o1` set pin A3 to output with bit value '1'  
__NB__: a line shows the port name with thereafter the value of port bit 0 at index 0.  
__NNB__: No peripheral function is changed with this command. The command test is meant for that.  


<a name="test"></a>
### test
__test__ *subcommand* ?*arg ...*?  
Not implemented yet.

<a name="var"></a>
### var
__var__ *vbe* ?*value ...*?  
SAM1 tester variables consist of settings and other values that can be accessed through  
numbered identifiers. The unique identifier of a variable is called its `infonr`. An infonr for  
settings is a negative number and an infonr for other values is a non-negative number.  
Because the infonr of a variable can change when the software gets an update, they are  
gathered in variable groups that can be identified with a group number `groupnr`.  
A groupnr for settings is a negative number and a groupnr for other variables is non-negative.  
Groupnr -1 has a special meaning, it identifies the device and software and therefore its  
values are text formatted before being output and reset to their default values after a device  
reset. Accessing variables with their infonr will end at the end of their variable group.  
The *vbe*-values begin and end default to the indices within the groupnr given before the point, or  
to the indices within the groupnr containing the infonr with the entered value without point.  
__todo:__ __.__*begin*__.__*end* acting on infonr when no groupnr is entered.  
*examples:*  
`var -4` show the value of the variable with infonr -4 upto the end of its group.  
`var -4 0` set the value of the variable with infonr -4 to 0.  
`var -` show all infonr with negative index, grouped per line.  
`var +` is the same for all infonr with non-negative index.  
`var 2.` show all variables of groupnr 2,  
`var 2.0.1 3 2 1` sets the value of the first three vars in groupnr 2 to 3 and 2 and 1 respectively.  
`var 11 1 2 3 4 5 6 7 8 9 10` set variable with infonr 11 to 1, the next to 2, and so on upto the  
value of 10 or the end of the group containing infonr 9.  
