COMM32.PAS
==========

Version 1.00

Comm32.pas is a simple Communications VC for Borland Delphi 2.0 which 
demonstrates the Win32 Communications functions and the new Delphi
'TThread' class. It is implemented using two threads: one for reading
from, and one for writing to a Comm Port.

It probably needs to be implemented as a single thread for read/write
operations if it is needed for any synchronisation functions (such as
file-transfer algorithms).

I started to create a 'TAPI' component to use in conjunction with this
component (hence the 'OnRequestHangup' property) but as of this version
it is incomplete.

Version 1.02 - by small-pig team

    1. Add Read/Write timing control
    2. Add Data bits, Parity, Stop bits properties
    3. Support software and hardware flow control: DTR/DSR, CTS/CTS, XON/XOFF
    4. Add 'Sender' parameter in OnReceiveData

Version 2.0 - - by small-pig team

    1. Support separatly DTR/DSR and RTS/CTS hardware flow control setting
    2. Support separatly OutX and InX software flow control setting
    3. Log file(for debug) may used by many comms at the same time
    4. Add DSR sensitivity property
    5. You can set the error char. replacement when parity error
    6. Let XonLim/XoffLim and XonChar/XoffChar setting by yourself
    7. You may change flow-control when comm is still opened
    8. Change TComm32 to TComm
    9. Add OnReceiveError event handler when overrun, framing error,
       parity error
    10. Fix some bug

Version 2.01 - - by small-pig team

    1. Support some property about modem.
    2. Add OnModemStateChange event hander when RLSD(CD) change state

Version 2.02 - - by small-pig team

    1. Bug fix: When receive XOFF character, the system FAULT!!!!
    2. Remove CommFileLog property

SPCOMM.PAS
==========

Version 2.5

    1. Add OnSendDataEmpty event handler when all data in buffer
       are sent(send-buffer become empty) this handler is called.
       You may call send data here.
    2. Change the ModemState parameters in OnModemStateChange
       to ModemEvent to indicate what modem event make this call
    3. Add RING signal detect. When RLSD changed state or
       RING signal was detected, OnModemStateChange handler is called
    4. Change XonLim and XoffLim from 100 to 500
    5. Remove TWriteThread.WriteData member
    6. PostHangupCall is re-design for debuging function
    7. Add a boolean property SendDataEmpty, True when send buffer
       is empty


USAGE
=====

To use the component once it is installed:
  1)  Attach an event handler to 'OnReceiveData'.
  2)  Call 'StartComm' to open the port.
  3)  Use the 'WriteCommData' method to write to the Comm port.
  4)  Call 'StopComm' to close the port.


PROPERTY
========

  CommName : String
    
      The name of comm port. The comm port is named 'COM1', 'COM2',...
      This comm port must exist when you open it. 

  BaudRate : DWORD

      The baud rate for this comm port. It must be a legal value for your
      serial port can accept it.
      You can change this value when the comm is open and
      the real baud rate is changed immediately.

  ParityCheck : Boolean

      Specifies whether parity checking is enabled. If this member is TRUE,
      parity checking is performed and errors are reported
      (to OnReceiveError handler).
      You can change this value when the comm is open.

  Outx_CtsFlow : Boolean

      Specifies whether the CTS (clear-to-send) signal is monitored
      for output flow control. If this member is TRUE and CTS is turned off,
      output is suspended until CTS is sent again.
      CTS is a input pin. You can read its state from MSR register. It
      usually connect to RTS pin in the other end.
      It is often used for hardware flow control to indicate that the other
      end if being waiting for data.
      You can change this value when the comm is open.

      CTS  Input on   | the other end is waiting for data
          ------------+----------------------------------------
           Input off  | the other end will NOT receive any data

  Outx_DsrFlow : Boolean

      Specifies whether the DSR (data-set-ready) signal is monitored for
      output flow control. If this member is TRUE and DSR is turned off,
      output is suspended until DSR is sent again.
      DSR is a input pin. You can read its state from MSR register. It
      usually connect to DTR pin in the other end.
      It is often used for hardware flow control to indicate that the other
      end is working(active, ready, wait for data...)
      You can change this value when the comm is open.

      DSR  Input on   | the other end is ready, and wait for your data
          ------------+----------------------------------------------------
           Input off  | the other end is not ready, we cannot send data out

  DtrControl : ( DtrEnable, DtrDiable, DtrHandshake )

      Specifies the DTR (data-terminal-ready) flow control. This member
      can be one of the following values:

      Value          Meaning
      -------------  ------------------------------------------------------
      DtrDiable      Disables the DTR line when the device is
                     opened and leaves it disabled.
      DtrEnable      Enables the DTR line when the device is
                     opened and leaves it on.
      DtrHandshake   Enables DTR handshaking. When our comm port is opened
                     and ready for receiving data, DTR is enabled. When
                     our comm port is suspend, closed or has error
                     (and not cleared), DTR is disable.
                     If handshaking is enabled, it is an error for the
                     application to adjust the line by using the Win32
                     EscapeCommFunction function.

      DTR is a output pin. You can set its state in MCR register.
      It usually connect to DSR pin in the other end.
      You can change this value when the comm is open.

  DsrSensitivity : Boolean;

      Specifies whether the communications driver is sensitive to the
      state of the DSR signal. If this member is TRUE, the driver ignores
      any bytes received, unless the DSR modem input line is high.
      You can change this value when the comm is open.

       TRUE  | ignore any received byte
      -------+------------------------------
       FALSE | nothing

  TxContinueOnXoff : Boolean;

      Specifies whether transmission stops when the input buffer is full
      and the driver has transmitted the XoffChar character. If this member
      is TRUE, transmission continues after the input buffer has come within
      XoffLim bytes of being full and the driver has transmitted the
      XoffChar character to stop receiving bytes. If this member is FALSE,
      transmission does not continue until the input buffer is within XonLim
      bytes of being empty and the driver has transmitted the XonChar
      character to resume reception.
      You can change this value when the comm is open.

       TRUE  | enable or disable transmisstion has no any relation with
             | input buffer
      -------+-------------------------------------------------------------
       FALSE | When Xoff is send(input buffer will be full),stop transmission.
             | And stop continue until Xon is send(input buffer will empty).

  Outx_XonXoffFlow : Boolean

      Specifies whether XON/XOFF flow control is used during transmission.
      If this member is TRUE, transmission stops when the XoffChar character
      is received and starts again when the XonChar character is received.
      You can change this value when the comm is open.

       Action | Xoff received      | Xon received
      --------+--------------------+---------------------------
       TRUE   | stop transmission  | resume transmission
              | discard Xoff char  | discard Xon char
      --------+--------------------+------------------------------
       FALSE  | read in Xoff char  | read in Xon char

  Inx_XonXoffFlow : Boolean

      Specifies whether XON/XOFF flow control is used during reception.
      If this member is TRUE, the XoffChar character is sent when the
      input buffer comes within XoffLim bytes of being full, and the
      XonChar character is sent when the input buffer comes within XonLim
      bytes of being empty.
      You can change this value when the comm is open.

              |               input buffer comes within
              +-----------------------------+------------------------------
       Action | XoffLim bytes of being full | XonLim bytes of being empty
      --------+-----------------------------+------------------------------
       TRUE   | Xoff char send              | Xon char send
      --------+-----------------------------+------------------------------
       FALSE  | do nothing                  | do nothing

  ReplaceWhenParityError : Boolean

      Specifies whether bytes received with parity errors are replaced with
      the character specified by the ReplacedChar member. If this member is
      TRUE and the ParityCheck member is TRUE, replacement occurs. 
      You can change this value when the comm is open.

  IgnoreNullChar : Boolean

      Specifies whether null bytes are discarded. If this member is TRUE,
      null bytes are discarded when received.
      You can change this value when the comm is open.

  RtsControl : ( RtsEnable, RtsDiable, RtsHandshake, TransmissionAvailableControl );

      Specifies the RTS (request-to-send) flow control. If this value is zero, the default is RTS_CONTROL_HANDSHAKE. This member can be one of the following values:

      Value                 Meaning
      ------------------   -------------------------------------------------
      RtsDiable             Disables the RTS line when the device
                            is opened and leaves it disabled.
      RtsEnable             Enables the RTS line when the device
                            is opened and leaves it on.
      RtsHandshake          Enables RTS handshaking. The driver raises the
                            RTS line when the "type-ahead" (input) buffer is
                            less than one-half full and lowers the RTS line
                            when the buffer is more than three-quarters full.
                            If handshaking is enabled, it is an error for the
                            application to adjust the line by using the
                            Win32 EscapeCommFunction function.

                      input buffer | less than 1/2 full | more than 3/4 full
                     --------------+--------------------+--------------------
                        RTS        | output set         | output clear   

      TransmissionAvailable
                            Specifies that the RTS line will be enabled
                            if bytes are available for transmission. After
                            all buffered bytes have been sent, the RTS line
                            will be disable.

                      output buffer | have available byte | no any data
                     ---------------+---------------------+----------------- 
                        RTS         | output set          | output clear

      RTS is a output pin. You can set its state in MCR register.
      It usually connect to CTS pin in the other end.
      You can change this value when the comm is open.

  XonLimit : WORD

      Specifies the minimum number of bytes allowed in the input buffer
      before the XON character is sent.
      You can change this value when the comm is open.

      Example:
          The size of input buffer is 1000 bytes and XonLimit is 100.
          And XOff character have sent before.
          When bytes in input buffer from 101 to 100, the Xon character is
          sent.

  XoffLimit : WORD

      Specifies the maximum number of bytes allowed in the input buffer
      before the XOFF character is sent. The maximum number of bytes
      allowed is calculated by subtracting this value from the size,
      in bytes, of the input buffer.
      You can change this value when the comm is open.

      Example:
          The size of input buffer is 1000 bytes and XonLimit is 100.
          When bytes in input buffer from 899 to 900, the Xoff character is
          sent.

  ByteSize : ( _5, _6, _7, _8 )

      Specifies the number of bits in the bytes transmitted and received. 
      You can change this value when the comm is open.

  Parity : ( None, Odd, Even, Mark, Space )

      Specifies the parity scheme to be used. This member can be one of the
      following values:

      Value   Meaning
      ------- ---------------------
      None    No parity
      Odd     Sum of logic '1' bits (data bits and parity bit) is odd
      Even    Sum of logic '1' bits (data bits and parity bit) is even
      Mask    Parity bit is always in logic '1'
      Space   Parity bit is always in logic '0'

      You can change this value when the comm is open.


  StopBits : ( _1, _1_5, _2 )

      Specifies the number of stop bits to be used.
      You can change this value when the comm is open.

  XonChar : AnsiChar

      Specifies the value of the XON character for both transmission and
      reception.
      You can change this value when the comm is open.

  XoffChar : AnsiChar

      Specifies the value of the XOFF character for both transmission and
      reception.
      You can change this value when the comm is open.

  ReplacedChar : AnsiChar

      Specifies the value of the character used to replace bytes received
      with a parity error when ReplaceWhenParityError member is TRUE and
      ParityCheck member is TRUE.
      You can change this value when the comm is open.

  Handle : THandle   <read-only>

      The comm file handle. Get from 'CreateFile' function in 'StartComm'.
      You can use it for another Win32 COMM API operation

  SendDataEmpty : Boolean   <run-time & read-only>

      True when send-buffer is empty and no any pending send require in message
      queue. False if send-buffer is not empty or have send require in message
      queue.

METHOD
======

  procedure StartComm

      Start a communication for this comm port.
      If failure, raise ECommsError exception and ECommsError.Message contain
      following string:
          'This serial port already opened'
          'Error opening serial port'
          'File handle is not a comm handle'
          'Cannot setup comm buffer'
          'Unable to create event'
          'Unable to create read thread'
          'Unable to create write thread'

  procedure StopComm

      Stop and end all communication threads for this comm port.
      No any return.

  function WriteCommData( pDataToWrite: PChar;
                          dwSizeofDataToWrite: Word ): Boolean

      Send a String to the Write Thread to be written to the Comm.
      This subroutine will return immediately. The send operation will
      do in background.
      Parameters:
          pszStringToWrite     - string to Write to Comm port.
          nSizeofStringToWrite - length of pszStringToWrite.
      Return:
          TRUE : if the PostMessage to write thread is successful.
          FALSE: if PostMessage fails or Write thread doesn't exist.

  function GetModemState : DWORD;

     Get and return the modem state right now. They are the state of modem
     input pin from MSR.
     The return value can be one or more of the following codes:

      Value       Meaning
      ----------  -----------------------------------------------------------
      MS_CTS_ON   The CTS (clear-to-send) signal is on.
      MS_DSR_ON   The DSR (data-set-ready) signal is on.
      MS_RING_ON  The ring indicator signal is on.
      MS_RLSD_ON  The RLSD (receive-line-signal-detect) signal is on.

      The function fails if the hardware does not support the
      control-register values.

EVENT HANDLER
=============

  OnReceiveData : procedure (Sender: TObject;
                             Buffer: Pointer;
                             BufferLength: Word) of object

      When
        1. The input buffer contains received data and
        2. more than a limit size or
        3. read time-out
      the event handler is called
      Sender : point to TComm object which raise this call
      Buffer : the buffer which contains received data
      BufferLength : the size of received data in Buffer

  OnReceiveError : procedure(Sender: TObject; EventMask : DWORD) of object

      When error happend during receiving, the event handler is called.
      Sender : point to TComm object which raise this call
      EventMask : the value get from ClearCommError in 'HandleCommEvent'
                  A 32-bit variable to be filled with a mask indicating the
                  type of error. This parameter can be one or more of the
                  following error codes:

      Value       Meaning
      ----------  -----------------------------------------------------------
      CE_BREAK    The hardware detected a break condition. (no support now)
      CE_DNS      Windows 95 only: A parallel device is not selected.
      CE_FRAME    The hardware detected a framing error.
      CE_IOE      An I/O error occurred during communications with the device.
      CE_MODE     The requested mode is not supported, or the hFile parameter
                  is invalid. If this value is specified, it is the only
                  valid error.
      CE_OOP      Windows 95 only: A parallel device signaled that it is out
                  of paper.
      CE_OVERRUN  A character-buffer overrun has occurred. The next character
                  is lost.
      CE_PTO      Windows 95 only: A time-out occurred on a parallel device.
      CE_RXOVER   An input buffer overflow has occurred. There is either no
                  room in the input buffer, or a character was received after
                  the end-of-file (EOF) character.
                  (second condition is impossible happened under Win32)
      CE_RXPARITY The hardware detected a parity error. (no support)
      CE_TXFULL   The application tried to transmit a character, but the
                  output buffer was full. (no support)


  OnRequestHangup: procedure(Sender: TObject) of object;

      When the program is terminated abnormally, the event handler is
      called.

  OnModemStateChange : procedure(Sender: TObject; ModemEvent : DWORD) of object

      When Modem's RLSD(CD) change state (HIGH<->LOW) or the ring indicator
      signal is detected, this event handler is
      called.
      Sender : point to TComm object which raise this call
      ModemEvent : A 32-bit variable to be filled with a mask indicating
                   what made this call. This parameter can be one or more
                   of the following codes:

      Value       Meaning
      ----------  -----------------------------------------------------------
      ME_CTS      The CTS (clear-to-send) signal has changed state.
                  (support in future, not support now)
      ME_DSR      The DSR (data-set-ready) signal has changed state.
                  (support in future, not support now)
      ME_RING     The ring indicator signal was detected.
      ME_RLSD     The RLSD (receive-line-signal-detect) signal has changed
                  state.

      Call GetModemState method to get the real state of modem state.

LEGALITIES
==========

This component is totally free (along with source code).

Small-Pig Team
29/4/97
E-mail:  spigteam@vlsi.ice.cycu.edu.tw
