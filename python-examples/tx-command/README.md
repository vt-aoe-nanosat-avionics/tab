# Python TX Command

A Python script to transmit TAB commands over a serial connection

Usage:
* Ensure the dependencies outlined in the Python examples [README](../README.md)
  are installed.
* Ensure the [setup_dependencies.sh](../scripts/setup_dependencies.sh) has run
  successfully.
* Then execute the following `bash` commands:

```bash
source ../p3-env/bin/activate
python3 tx_command.py /dev/ttyUSB0
```

The ouput will then be:
```bash
>
```
This arrow is where the commands are typed. Once the command is entered a TAB frame is constructed and set via USART to the board, and a reply is shown. Many commands with payloads are not fully integrated into teh script format, and have hard coded payloads that can only be changed before running the script.
A few commands have examples of using the command and show the outputs from the commands. Each pair of lines shows what command was sent to the board and any payload that was associated with it. The second line shows the response that was recieved.

## Commands
 The Following Commands are valid however not every command is fully implemented

* [Commands](#commands): TAB commands
  * [Common Ack](#common-ack)
  * [Common Nack](#common-nack)
  * [Common Debug](#common-debug)
  * [Common Data](#common-data)
  * [Bootloader Ack](#bootloader-ack)
  * [Bootloader Nack](#bootloader-nack)
  * [Bootloader Ping](#bootloader-ping)
  * [Bootlaoder Erase](#bootloader-erase)
  * [Bootloader Write Page](#bootloader-write-page)
  * [Bootloader Write Page Addr32](#bootloader-write-page-addr32)
  * [Bootloader Jump](#bootloader-jump)
  * [Bootloader Power](#bootloader-power)
  * [App Get Telemetry](#app_get_telem)
  * [App Set Time](#app_set_time)
  * [App Get Time](#app_get_time)
  * [App Reboot](#app_reboot)
  * [Exit](#exit)
* [Protocol](#protocol): TAB protocol
* [License](#license)

These commands and the replies they generate are below

### <a name="common-ack"></a> Common Ack

This acknowledgement command is most often used as a reply indicating success.
It is also useful as a command to check whether the recipient is "alive."
* Name: `common_ack`
* Required parameters: None
* Optional parameters: None
* Reply: `common_ack`
* Example:
```bash
>common_ack
txcmd: common_ack hw_id:0x0012 msg_id:0x0000 src:gnd dst:cdh
reply: common_ack hw_id:0x0012 msg_id:0x0000 src:cdh dst:gnd
```


### <a name="common-nack"></a> Common Nack

This negative-acknowledgement command is used as a reply indicating failure.
* Name: `common_nack`
* Required parameters: None
* Optional parameters: None
* Reply: `common_nack`
  * Because this command is used as a reply, sending this command generates a
    common nack reply.


### <a name="common-debug"></a> Common Debug

This command supports a variable-length ASCII payload useful for debugging.
* Name: `common_debug`
* Required parameters: A sequence of one or more ASCII characters 
* Optional parameters: None
* Reply: `common_debug` with the original payload of ASCII characters
* Example:
```bash
>common_debug hello
txcmd: common_debug hw_id:0x0012 msg_id:0x0002 src:gnd dst:cdh "hello"
reply: common_nack hw_id:0x0012 msg_id:0x0002 src:cdh dst:gnd
```


### <a name="common-data"></a> Common Data

This command supports a variable-length byte payload useful for data transfer. Currently the bytes sent are hard coded and cannot be changed during runtime
* Name: `common_data`
* Required parameters: A sequence of one or more bytes
* Optional parameters: None
* Reply:
  * If the byte payload handler is successful: `common_ack`
  * Otherwise: `common_nack`


### <a name="bootloader-ack"></a> Bootloader Ack

The bootloader acknowledgement command is exclusively used as a bootloader reply
indicating success.
* Name: `bootloader_ack`
* Required parameters: None
* Optional parameters: Reason bytes
* Reply: `common_nack`
  * Because this command is used as a reply, sending this command generates a
    common nack reply.


### <a name="bootloader-nack"></a> Bootloader Nack

The bootloader negative-acknowledgement command is used as a bootloader reply
indicating failure.
* Name: `bootloader_nack`
* Required parameters: None
* Optional parameters: None
* Reply: `common_nack`
  * Because this command is used as a reply, sending this command generates a
    common nack reply.


### <a name="bootloader-ping"></a> Bootloader Ping

The bootloader ping command checks whether the bootloader is active.
* Name: `bootloader_ping`
* Required parameters: None
* Optional parameters: None
* Reply:
  * If the bootloader is active: `bootloader_ack` with the PONG payload
  * Otherwise: `common_nack`


### <a name="bootloader-erase"></a> Bootloader Erase

This command instructs the bootloader to erase all applications.
* Name: `bootloader_erase`
* Required parameters: None
* Optional parameters: Status
* Reply:
  * If the bootloader is active and successfully performs the erase:
    `bootloader_ack` with the ERASED payload
  * If the bootloader is active and fails to perform the erase:
    `bootloader_nack`
  * Otherwise: `common_nack`


### <a name="bootloader-write-page"></a> Bootloader Write Page

The bootloader write page command instructs the bootloader to write 128 bytes to
a 128-byte region of (presumably nonvolatile) memory indexed by the "page
number" parameter (the "sub-page ID"). For backwards-compatibility, the 128
bytes of "sub-page" data are optional (presumably, this feature allows for
"dry-runs" or page number validity probes). The "page number" parameter
("sub-page ID") is required.
* Name: `bootloader_write_page`
* Required parameters: Page Number (Sub-Page ID)
* Optional parameters: (Sub-)Page Data
* Reply:
  * If the bootloader is active and successfully performs the write:
    `bootloader_ack` with the Page Number as the payload
  * If the bootloader is active and fails to perform the write:
    `bootloader_nack`
  * Otherwise: `common_nack`


### <a name="bootloader-write-page-addr32"></a> Bootloader Write Page Addr32

This command instructs the bootloader to write 128 bytes to a region of
(presumably nonvolatile) memory starting at the provided 32-bit address. The
target start address is required, and the 128 bytes of data are optional. This
feature allows for "dry-runs" or target start address validity probes.
* Name: `bootloader_write_page_addr32`
* Required parameters: Start Address
* Optional parameters: Sub-Page Data
* Reply:
  * If the bootloader is active and successfully performs the write:
    `bootloader_ack` with the Start Address as the payload
  * If the bootloader is active and fails to perform the write:
    `bootloader_nack`
  * Otherwise: `common_nack`


### <a name="bootloader-jump"></a> Bootloader Jump

The bootloader jump command instructs the bootloader to start a user
application.
* Name: `bootloader_jump`
* Required parameters: None
* Optional parameters: None
* Reply:
  * If the bootloader is active and successfully performs the jump:
    `bootloader_ack` with the JUMPED payload
  * If the bootloader is active and fails to perform the jump:
    `bootloader_nack`
  * Otherwise: `common_nack`


### <a name="bootloader-power"></a> Bootloader Power

This command instructs the bootloader to change the power mode it is operating in
* Name: `bootloader_power`
* Required parameters: Power Mode
* Optional parameters: None
* Reply:
  * If the bootloader is active and successfully changes power mode:
    `bootloader_ack`
  * If the bootloader is active and fails to change power mode:
    `bootloader_nack`
  * If an invalid power transition is requested:
    `bootloader_nack`
  * Valid mode transitions:
    ![state transition diagram!](../../images/state_transition_diagram.png)
* Example:
```bash
>bootloader_power sleep
txcmd: bootloader_power hw_id:0x0012 msg_id:0x0001 src:gnd dst:cdh
reply: bootloader_ack hw_id:0x0012 msg_id:0x0001 src:cdh dst:gnd
```


### <a name="app_get_telem"></a> App Get Telemetry

This command has an unknown function
* Name: `app_get_telem`
* Required parameters: None
* Optional parameters: None
* Reply: `app_telem`


### <a name="app_set_time"></a> App Set Time

This command sets the current time as seconds since the J2000 epoch. The seconds are calculated based on the current machine time running this python script
* Name: `app_set_time`
* Required parameters: Seconds and Nanoseconds
* Optional parameters: None
* Reply:
  * If the time is set
    `common_ack`
  * If the time is not set
    `common_nack`
* Example:
```bash
>app_set_time
txcmd: app_set_time hw_id:0x0012 msg_id:0x0002 src:gnd dst:cdh sec:781284529 ns:133918000
reply: common_ack hw_id:0x0012 msg_id:0x0002 src:cdh dst:gnd
```


### <a name="app_get_time"></a> App Get Time

This command gets the current time as seconds since the J2000 epoch
* Name: `app_get_time`
* Required parameters: None
* Optional parameters: None
* Reply:
  * If the time is not set
    `common_nack`
  * If time is set: `app_set_time` with the seconds and nanoseconds as the payload
```bash
>app_get_time
txcmd: app_get_time hw_id:0x0012 msg_id:0x0003 src:gnd dst:cdh
reply: app_set_time hw_id:0x0012 msg_id:0x0003 src:cdh dst:gnd sec:781284587 ns:184000000
```


### <a name="app_reboot"></a> App Reboot

This command is meant to reboot the ta-expt board, but is currently unimplemented in the tx_command script
* Name: `app_reboot`
* Required parameters: None
* Optional parameters: Delay (seconds)
* Reply:
  * If there is no delay or the delay is less than maximum
    `common_ack`
  * If the delay is more than the maximum value
    `common_nack`

### <a name="exit"></a> Exit
This command exits from the tx_command script
* Name: `exit`
* Required parameters: None
* Optional parameters: None
* Reply: None




## <a name="license"></a> License

Written by Andrew McGrellis  
Other contributors: None

See the top-level LICENSE file for the license.

