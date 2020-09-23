<div align="center">

## How to Write Your Own GPS Applications: Part 1


</div>

### Description

What is it that GPS applications need to be good enough for in-car navigation? Also, how does the process of interpreting GPS data actually work? In this two-part series, I will cover both topics and give you the skills you need to write a commercial-grade GPS application that works with a majority of GPS devices in the industry today.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Jon Person \(GeoFrameworks\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/jon-person-geoframeworks.md)
**Level**          |Beginner
**User Rating**    |4.9 (264 globes from 54 users)
**Compatibility**  |C\#, VB\.NET, ASP\.NET
**Category**       |[Complete Applications](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/complete-applications__10-7.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/jon-person-geoframeworks-how-to-write-your-own-gps-applications-part-1__10-2714/archive/master.zip)





### Source Code

<P><STRONG><EM>“I am continually amazed by how little code is required to use
atomic clocks in satellites 11,000 miles above my head.”</EM></STRONG></P>
<H2>Introduction</H2>
<P>    What is it that GPS applications need to be good
enough to use in a commercial environment, such as in-car navigation? 
Also, how does the process of interpreting GPS data actually work?  In this
two-part series, I will cover both topics and give you the skills you need to
write a commercial-grade GPS application that works with a majority of GPS
devices in the industry today.</P>
<H2>One Powerful Sentence</H2>
<P>    This first part in the series will explore the task
of interpreting raw GPS data.  Fortunately, the task is simplified thanks
to the National Marine Electronics Association (www.nmea.org) which introduced a
standard for the industry now in use by a vast majority of GPS devices.  To
give developers a head start, I chose to use some Visual Studio.NET source code
from my “GPS.NET Global Positioning SDK” component.  (The code is stripped
of features like multithreading and error handling for
brevity.)<BR>     NMEA data is sent as comma-delimited
“sentences” which contain information based on the first word of the
sentence.  There are over fifty kinds of sentences, yet an interpreter
really only needs to handle a few to get the job done.   The most common
NMEA sentence of all is the “Recommended Minimum” sentence, which begins with
“$GPRMC.”  Here is an example:</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P> <FONT
face=courier>$GPRMC,040302.663,A,3939.7,N,10506.6,W,0.27,358.86,200804,,*1A</FONT></P></BLOCKQUOTE>
<P>      This one sentence contains nearly everything a
GPS application needs: latitude, longitude, speed, bearing, satellite-derived
time, fix status and magnetic variation.  </P>
<H2>The Core of An Interpreter</H2>     The first step in
making an NMEA interpreter is writing a method which does two things: separating
each sentence into its individual words and examining the first word to figure
out what information is available to extract.  Listing 1-1 shows the start
of the interpreter class.
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM> (Listing 1-1: The core of an NMEA interpreter is a function which
divides NMEA sentences into individual words.)</EM> <PRE lang=vb.net>'*******************************************************
'** Listing 1-1. The core of an NMEA interpreter
'*******************************************************
Public Class NmeaInterpreter
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Look at the first word to decide where to go next
 Select Case Words(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  ' Indicate that the sentence was recognized
  Return True
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<P>     The next step is to perform actual extraction of
information, starting with latitude and longitude.  Latitude and longitude
are stored in the form “DDD°MM’SS.S,” where D represents hours (also called
“degrees”), M represents minutes and S represents seconds.  Coordinates can
be displayed in shorthand, such as “DD°MM.M’” or even “DD°.”  The fourth
word in the sentence, “3939.7,” shows the current latitude as hours and minutes
(39°39.7’), except the numbers are squished together.  The first two
characters (39) represent hours and the remainder of the word (39.7) represents
minutes.  Longitude is structured the same way, except that the first three
characters represent hours (105°06.6’).  Words five and seven indicate the
“hemisphere,” where “N” means “North,” “W” means “West” etc.  The
hemisphere is appended to the end of the numeric portion to make a complete
measurement.<BR>     I’ve found that NMEA interpreters are
much easier to work with they are event-driven.  This is because data
arrives in no particular order.   An event-driven class gives the
interpreter the most flexibility and responsiveness to an application.  So,
I’ll design the interpreter to report information using events.  The first
event, PositionReceived, will be raised whenever the current latitude and
longitude are received.  Listing 1-2 expands the interpreter to report the
current position.</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM> (Listing 1-2: The interpreter can now report the current latitude
and longitude.)</EM> <PRE lang=vb.net>'*******************************************************
'** Listing 1-2. Extracting information from a sentence
'*******************************************************
Public Class NmeaInterpreter
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<P>     One thing to watch out for here is that some GPS
devices will report blank values when no information is known.  Therefore,
it’s a good idea to test each word for a value before parsing.  If you need
to type the degree symbol (°), hold down the Alt key and type “0176” on the
numeric keypad.</P>
<H2>Taking Out the Garbage</H2>
<P>     A checksum is calculated as the XOR of bytes between
(but not including) the dollar sign and asterisk.  This checksum is then
compared with the checksum from the sentence.  If the checksums do not
match, the sentence is typically discarded.  This is okay to do because the
GPS devices tend to repeat the same information every few seconds.  With
the ability to compare checksums, the interpreter is able to throw out any
sentence with an invalid checksum.  Listing 1-3 expands the interpreter to
do this.</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM> (Listing 1-3:  The interpreter can now detect errors and parse
only error-free NMEA data.)</EM> <PRE lang=vb.net>'*******************************************************
'** Listing 1-3. Detecting and handling NMEA errors
'*******************************************************
Public Class NmeaInterpreter
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Discard the sentence if its checksum does not match our calculated checksum
 If Not IsValid(sentence) Then Return False
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Returns True if a sentence's checksum matches the calculated checksum
 Public Function IsValid(ByVal sentence As String) As Boolean
 ' Compare the characters after the asterisk to the calculation
 Return sentence.Substring(sentence.IndexOf("*") + 1) = GetChecksum(sentence)
 End Function
 ' Calculates the checksum for a sentence
 Public Function GetChecksum(ByVal sentence As String) As String
 ' Loop through all chars to get a checksum
 Dim Character As Char
 Dim Checksum As Integer
 For Each Character In sentence
  Select Case Character
  Case "$"c
   ' Ignore the dollar sign
  Case "*"c
   ' Stop processing before the asterisk
   Exit For
  Case Else
   ' Is this the first value for the checksum?
   If Checksum = 0 Then
   ' Yes. Set the checksum to the value
   Checksum = Convert.ToByte(Character)
   Else
   ' No. XOR the checksum with this character's value
   Checksum = Checksum Xor Convert.ToByte(Character)
   End If
  End Select
 Next
 ' Return the checksum formatted as a two-character hexadecimal
 Return Checksum.ToString("X2")
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<H2>Wireless Atomic Time</H2>
<P>     Time is the cornerstone of GPS technology because
distances are measured at the speed of light.  Each GPS satellite contains
four atomic clocks which it uses to time its radio transmissions within a few
nanoseconds.  One fascinating feature is that with just a few lines of
code, these atomic clocks can be used to synchronize a computer’s clock with
millisecond accuracy.  The second word of the $GPRMC sentence,
“040302.663,” contains satellite-derived time in a compressed format.  The
first two characters represent hours, the next two represent minutes, the next
two represent seconds, and everything after the decimal place is
milliseconds.   So, the time is 4:03:02.663 AM.  However,
satellites report time in universal time (GMT+0), so the time must to be
adjusted to the local time zone.   Listing 1-4 adds support for
satellite-derived time and uses the DateTime.ToLocalTime method to convert
satellite time to the local time zone.</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM> (Listing 1-4: This class can now use atomic clocks to synchronize
your computer’s clock wirelessly.)</EM> <PRE lang=vb.net>'********************************************************
'** Listing 1-4. Add support for satellite-derived time
'********************************************************
Public Class NmeaInterpreter
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 Public Event DateTimeChanged(ByVal dateTime As DateTime)
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Discard the sentence if its checksum does not match our calculated checksum
 If Not IsValid(sentence) Then Return False
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Do we have enough values to parse satellite-derived time?
 If Words(1) <> "" Then
  ' Yes. Extract hours, minutes, seconds and milliseconds
  Dim UtcHours As Integer = CType(Words(1).Substring(0, 2), Integer)
  Dim UtcMinutes As Integer = CType(Words(1).Substring(2, 2), Integer)
  Dim UtcSeconds As Integer = CType(Words(1).Substring(4, 2), Integer)
  Dim UtcMilliseconds As Integer
  ' Extract milliseconds if it is available
  If Words(1).Length > 7 Then UtcMilliseconds = CType(Words(1).Substring(7), Integer)
  ' Now build a DateTime object with all values
  Dim Today As DateTime = System.DateTime.Now.ToUniversalTime
  Dim SatelliteTime As New System.DateTime(Today.Year, Today.Month, _
  Today.Day, UtcHours, UtcMinutes, UtcSeconds, UtcMilliseconds)
  ' Notify of the new time, adjusted to the local time zone
  RaiseEvent DateTimeChanged(SatelliteTime.ToLocalTime)
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Returns True if a sentence's checksum matches the calculated checksum
 Public Function IsValid(ByVal sentence As String) As Boolean
 ' Compare the characters after the asterisk to the calculation
 Return sentence.Substring(sentence.IndexOf("*") + 1) = GetChecksum(sentence)
 End Function
 ' Calculates the checksum for a sentence
 Public Function GetChecksum(ByVal sentence As String) As String
 ' Loop through all chars to get a checksum
 Dim Character As Char
 Dim Checksum As Integer
 For Each Character In sentence
  Select Case Character
  Case "$"c
   ' Ignore the dollar sign
  Case "*"c
   ' Stop processing before the asterisk
   Exit For
  Case Else
   ' Is this the first value for the checksum?
   If Checksum = 0 Then
   ' Yes. Set the checksum to the value
   Checksum = Convert.ToByte(Character)
   Else
   ' No. XOR the checksum with this character's value
   Checksum = Checksum Xor Convert.ToByte(Character)
   End If
  End Select
 Next
 ' Return the checksum formatted as a two-character hexadecimal
 Return Checksum.ToString("X2")
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<H2>Direction & Speed Alerts </H2>
<P>     GPS devices analyze your position over time to
calculate speed and bearing.  The $GPRMC sentence at the beginning of this
article also includes these readings.  Speed is always reported in knots
and bearing is reported as an “azimuth,” a measurement around the horizon
measured clockwise from 0° to 360° where 0° represents north, 90° means east,
and etc.  A little math is applied to convert knots into miles per
hour.   The power of GPS is again demonstrated with one line of code
in listing 1-5 which figures out if a car is over the speed limit.  </P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P> <EM>(Listing 1-5: This class can now tell you which direction you’re
going and help prevent a speeding ticket.)</EM> <PRE lang=vb.net>'*******************************************************
'** Listing 1-5. Extracting speed and bearing
'*******************************************************
Public Class NmeaInterpreter
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 Public Event DateTimeChanged(ByVal dateTime As DateTime)
 Public Event BearingReceived(ByVal bearing As Double)
 Public Event SpeedReceived(ByVal speed As Double)
 Public Event SpeedLimitReached()
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Discard the sentence if its checksum does not match our calculated checksum
 If Not IsValid(sentence) Then Return False
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Do we have enough values to parse satellite-derived time?
 If Words(1) <> "" Then
  ' Yes. Extract hours, minutes, seconds and milliseconds
  Dim UtcHours As Integer = CType(Words(1).Substring(0, 2), Integer)
  Dim UtcMinutes As Integer = CType(Words(1).Substring(2, 2), Integer)
  Dim UtcSeconds As Integer = CType(Words(1).Substring(4, 2), Integer)
  Dim UtcMilliseconds As Integer
  ' Extract milliseconds if it is available
  If Words(1).Length > 7 Then UtcMilliseconds = CType(Words(1).Substring(7), Integer)
  ' Now build a DateTime object with all values
  Dim Today As DateTime = System.DateTime.Now.ToUniversalTime
  Dim SatelliteTime As New System.DateTime(Today.Year, Today.Month, _
  Today.Day, UtcHours, UtcMinutes, UtcSeconds, UtcMilliseconds)
  ' Notify of the new time, adjusted to the local time zone
  RaiseEvent DateTimeChanged(SatelliteTime.ToLocalTime)
 End If
 ' Do we have enough information to extract the current speed?
 If Words(7) <> "" Then
  ' Yes. Convert it into MPH
  Dim Speed As Double = CType(Words(7), Double) * 1.150779
  ' If we're over 55MPH then trigger a speed alarm!
  If Speed > 55 Then RaiseEvent SpeedLimitReached()
  ' Notify of the new speed
  RaiseEvent SpeedReceived(Speed)
 End If
 ' Do we have enough information to extract bearing?
 If Words(8) <> "" Then
  ' Indicate that the sentence was recognized
  Dim Bearing As Double = CType(Words(8), Double)
  RaiseEvent BearingReceived(Bearing)
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Returns True if a sentence's checksum matches the calculated checksum
 Public Function IsValid(ByVal sentence As String) As Boolean
 ' Compare the characters after the asterisk to the calculation
 Return sentence.Substring(sentence.IndexOf("*") + 1) = GetChecksum(sentence)
 End Function
 ' Calculates the checksum for a sentence
 Public Function GetChecksum(ByVal sentence As String) As String
 ' Loop through all chars to get a checksum
 Dim Character As Char
 Dim Checksum As Integer
 For Each Character In sentence
  Select Case Character
  Case "$"c
   ' Ignore the dollar sign
  Case "*"c
   ' Stop processing before the asterisk
   Exit For
  Case Else
   ' Is this the first value for the checksum?
   If Checksum = 0 Then
   ' Yes. Set the checksum to the value
   Checksum = Convert.ToByte(Character)
   Else
   ' No. XOR the checksum with this character's value
   Checksum = Checksum Xor Convert.ToByte(Character)
   End If
  End Select
 Next
 ' Return the checksum formatted as a two-character hexadecimal
 Return Checksum.ToString("X2")
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<H2>Are We Fixed Yet?</H2>
<P>     The $GPRMC sentence includes a value which indicates
whether or not a “fix” has been obtained.  A fix is possible when the
signal strength of at least three satellites is strong enough to be involved in
calculating your location.  If at least four satellites are involved,
altitude also becomes known.  The third word of the $GPRMC sentence is one
of two letters: “A” for “active,” where a fix is obtained, or “V” for “invalid”
where no fix is present.  Listing 1-6 includes code to examine this
character and report on fix status.  </P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM> (Listing 1-6: The interpreter now knows when the device has
obtained a fix.)</EM> <PRE lang=vb.net>'*******************************************************
'** Listing 1-6. Extracting satellite fix status
'*******************************************************
Public Class NmeaInterpreter
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 Public Event DateTimeChanged(ByVal dateTime As DateTime)
 Public Event BearingReceived(ByVal bearing As Double)
 Public Event SpeedReceived(ByVal speed As Double)
 Public Event SpeedLimitReached()
 Public Event FixObtained()
 Public Event FixLost()
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Discard the sentence if its checksum does not match our calculated checksum
 If Not IsValid(sentence) Then Return False
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Do we have enough values to parse satellite-derived time?
 If Words(1) <> "" Then
  ' Yes. Extract hours, minutes, seconds and milliseconds
  Dim UtcHours As Integer = CType(Words(1).Substring(0, 2), Integer)
  Dim UtcMinutes As Integer = CType(Words(1).Substring(2, 2), Integer)
  Dim UtcSeconds As Integer = CType(Words(1).Substring(4, 2), Integer)
  Dim UtcMilliseconds As Integer
  ' Extract milliseconds if it is available
  If Words(1).Length > 7 Then UtcMilliseconds = CType(Words(1).Substring(7), Integer)
  ' Now build a DateTime object with all values
  Dim Today As DateTime = System.DateTime.Now.ToUniversalTime
  Dim SatelliteTime As New System.DateTime(Today.Year, Today.Month, _
  Today.Day, UtcHours, UtcMinutes, UtcSeconds, UtcMilliseconds)
  ' Notify of the new time, adjusted to the local time zone
  RaiseEvent DateTimeChanged(SatelliteTime.ToLocalTime)
 End If
 ' Do we have enough information to extract the current speed?
 If Words(7) <> "" Then
  ' Yes. Convert it into MPH
  Dim Speed As Double = CType(Words(7), Double) * 1.150779
  ' If we're over 55MPH then trigger a speed alarm!
  If Speed > 55 Then RaiseEvent SpeedLimitReached()
  ' Notify of the new speed
  RaiseEvent SpeedReceived(Speed)
 End If
 ' Do we have enough information to extract bearing?
 If Words(8) <> "" Then
  ' Indicate that the sentence was recognized
  Dim Bearing As Double = CType(Words(8), Double)
  RaiseEvent BearingReceived(Bearing)
 End If
 ' Does the device currently have a satellite fix?
 If Words(2) <> "" Then
  Select Case Words(2)
  Case "A"
   RaiseEvent FixObtained()
  Case "V"
   RaiseEvent FixLost()
  End Select
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Returns True if a sentence's checksum matches the calculated checksum
 Public Function IsValid(ByVal sentence As String) As Boolean
 ' Compare the characters after the asterisk to the calculation
 Return sentence.Substring(sentence.IndexOf("*") + 1) = GetChecksum(sentence)
 End Function
 ' Calculates the checksum for a sentence
 Public Function GetChecksum(ByVal sentence As String) As String
 ' Loop through all chars to get a checksum
 Dim Character As Char
 Dim Checksum As Integer
 For Each Character In sentence
  Select Case Character
  Case "$"c
   ' Ignore the dollar sign
  Case "*"c
   ' Stop processing before the asterisk
   Exit For
  Case Else
   ' Is this the first value for the checksum?
   If Checksum = 0 Then
   ' Yes. Set the checksum to the value
   Checksum = Convert.ToByte(Character)
   Else
   ' No. XOR the checksum with this character's value
   Checksum = Checksum Xor Convert.ToByte(Character)
   End If
  End Select
 Next
 ' Return the checksum formatted as a two-character hexadecimal
 Return Checksum.ToString("X2")
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<P>     As you can see, a whole lot of information is packed
into a single NMEA sentence.  Now that the $GPRMC sentence has been fully
interpreted, the interpreter can be expanded to support a second sentence:
$GPGSV.  This sentence describes the configuration of satellites overhead,
in real-time.</P>
<H2>Real-Time Satellite Tracking</H2>
<P>     Knowing the location of satellites is important when
determining how precise readings are and how stable a GPS fix is.  
Since GPS precision will be covered in detail in part two of this series, so
this section will focus on interpreting satellite location and signal
strength.<BR>     There are twenty-four operational
satellites in orbit.  Satellites are spaced in orbit so that at any time a
minimum of six satellites will be in view to users anywhere in the world. 
Satellites are constantly in motion, which is good because it prevents the
existence of “blind spots” in the world with little or no satellite
visibility.  Just like finding stars in the sky, satellite locations are
described as the combination of an azimuth and an elevation.  As mentioned
above, azimuth measures a direction around the horizon.  Elevation measures
a degree value up from the horizon between 0° and 90°, where 0° represents the
horizon and 90° represents “zenith,” directly overhead.  So, if the device
says a satellite’s azimuth is 45° and its elevation is 45°, the satellite is
located halfway up from the horizon towards the northeast.  In addition to
location, devices report each satellite’s “Pseudo-Random Code” (or PRC) which is
a number used to uniquely identify one satellite from another. 
<BR>     Here’s an example of a $GPGSV sentence:</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><FONT
face=courier>$GPGSV,3,1,10,24,82,023,40,05,62,285,32,01,62,123,00,17,59,229,28*70</FONT></P></BLOCKQUOTE>
<P>     Each sentence contains up to four blocks of
satellite information, comprised of four words.  For example, the first
block is “24,82,023,40” and the second block is “05,62,285,32” and so on. 
The first word of each block gives the satellite’s PRC.  The second word
gives each satellite’s elevation, followed by azimuth and signal strength. 
If this satellite information were to be shown graphically, it would look like
figure 1-1.</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><BR><EM>(Images restricted. Go to http://www.gpsdotnet.com/images/Figure11.jpg for figure 1-1)<BR>(Figure
1-1: Graphical representation of a $GPGSV sentence, where the center of the
circle marks the current position and the edge of the circle marks the
horizon.)</EM></P></BLOCKQUOTE>
<P>     Perhaps the most important number in this sentence
is the “signal-to-noise ratio” (or SNR for short).  This number indicates
how strongly a satellite’s radio signal is being received.  Remember,
satellites transmit signals at the same strength, but things like trees and
walls can obscure a signal beyond recognition.  Typical SNR values are
between zero and fifty, where fifty means an excellent signal.  (SNR can be
as high as ninety-nine, but I’ve never seen readings above fifty even in wide
open sky.)  In Figure 1-1, the green satellites indicate a strong signal,
whereas the yellow satellite signifies a moderate signal (in part two, I will
provide a way to classify signal strengths).  Satellite #1’s signal is
completely obscured.  Listing 1-7 shows the interpreter after it is
expanded to read satellite info.</P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM>(Listing 1-7: The interpreter is improved to interpret the location of
GPS satellites currently in view.)</EM> <PRE lang=vb.net>'*******************************************************
'** Listing 1-7. Extracting satellite information
'*******************************************************
Public Class NmeaInterpreter
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 Public Event DateTimeChanged(ByVal dateTime As DateTime)
 Public Event BearingReceived(ByVal bearing As Double)
 Public Event SpeedReceived(ByVal speed As Double)
 Public Event SpeedLimitReached()
 Public Event FixObtained()
 Public Event FixLost()
 Public Event SatelliteReceived(ByVal pseudoRandomCode As Integer, _
 ByVal azimuth As Integer, _
 ByVal elevation As Integer, _
 ByVal signalToNoiseRatio As Integer)
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Discard the sentence if its checksum does not match our calculated checksum
 If Not IsValid(sentence) Then Return False
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case "$GPGSV"  ' A "Satellites in View" message was found
  Return ParseGPGSV(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Do we have enough values to parse satellite-derived time?
 If Words(1) <> "" Then
  ' Yes. Extract hours, minutes, seconds and milliseconds
  Dim UtcHours As Integer = CType(Words(1).Substring(0, 2), Integer)
  Dim UtcMinutes As Integer = CType(Words(1).Substring(2, 2), Integer)
  Dim UtcSeconds As Integer = CType(Words(1).Substring(4, 2), Integer)
  Dim UtcMilliseconds As Integer
  ' Extract milliseconds if it is available
  If Words(1).Length > 7 Then UtcMilliseconds = CType(Words(1).Substring(7), Integer)
  ' Now build a DateTime object with all values
  Dim Today As DateTime = System.DateTime.Now.ToUniversalTime
  Dim SatelliteTime As New System.DateTime(Today.Year, Today.Month, _
  Today.Day, UtcHours, UtcMinutes, UtcSeconds, UtcMilliseconds)
  ' Notify of the new time, adjusted to the local time zone
  RaiseEvent DateTimeChanged(SatelliteTime.ToLocalTime)
 End If
 ' Do we have enough information to extract the current speed?
 If Words(7) <> "" Then
  ' Yes. Convert it into MPH
  Dim Speed As Double = CType(Words(7), Double) * 1.150779
  ' If we're over 55MPH then trigger a speed alarm!
  If Speed > 55 Then RaiseEvent SpeedLimitReached()
  ' Notify of the new speed
  RaiseEvent SpeedReceived(Speed)
 End If
 ' Do we have enough information to extract bearing?
 If Words(8) <> "" Then
  ' Indicate that the sentence was recognized
  Dim Bearing As Double = CType(Words(8), Double)
  RaiseEvent BearingReceived(Bearing)
 End If
 ' Does the device currently have a satellite fix?
 If Words(2) <> "" Then
  Select Case Words(2)
  Case "A"
   RaiseEvent FixObtained()
  Case "V"
   RaiseEvent FixLost()
  End Select
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Interprets a "Satellites in View" NMEA sentence
 Public Function ParseGPGSV(ByVal sentence As String) As Boolean
 Dim PseudoRandomCode As Integer
 Dim Azimuth As Integer
 Dim Elevation As Integer
 Dim SignalToNoiseRatio As Integer
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Each sentence contains four blocks of satellite information. Read each block
 ' and report each satellite's information
 Dim Count As Integer
 For Count = 1 To 4
  ' Does the sentence have enough words to analyze?
  If (Words.Length - 1) >= (Count * 4 + 3) Then
  ' Yes. Proceed with analyzing the block. Does it contain any information?
  If Words(Count * 4) <> "" And Words(Count * 4 + 1) <> "" _
  And Words(Count * 4 + 2) <> "" And Words(Count * 4 + 3) <> "" Then
   ' Yes. Extract satellite information and report it
   PseudoRandomCode = CType(Words(Count * 4), Integer)
   Elevation = CType(Words(Count * 4 + 1), Integer)
   Azimuth = CType(Words(Count * 4 + 2), Integer)
   SignalToNoiseRatio = CType(Words(Count * 4 + 2), Integer)
   ' Notify of this satellite's information
   RaiseEvent SatelliteReceived(PseudoRandomCode, Azimuth, Elevation, _
   SignalToNoiseRatio)
  End If
  End If
 Next
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Returns True if a sentence's checksum matches the calculated checksum
 Public Function IsValid(ByVal sentence As String) As Boolean
 ' Compare the characters after the asterisk to the calculation
 Return sentence.Substring(sentence.IndexOf("*") + 1) = GetChecksum(sentence)
 End Function
 ' Calculates the checksum for a sentence
 Public Function GetChecksum(ByVal sentence As String) As String
 ' Loop through all chars to get a checksum
 Dim Character As Char
 Dim Checksum As Integer
 For Each Character In sentence
  Select Case Character
  Case "$"c
   ' Ignore the dollar sign
  Case "*"c
   ' Stop processing before the asterisk
   Exit For
  Case Else
   ' Is this the first value for the checksum?
   If Checksum = 0 Then
   ' Yes. Set the checksum to the value
   Checksum = Convert.ToByte(Character)
   Else
   ' No. XOR the checksum with this character's value
   Checksum = Checksum Xor Convert.ToByte(Character)
   End If
  End Select
 Next
 ' Return the checksum formatted as a two-character hexadecimal
 Return Checksum.ToString("X2")
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<H2>A World-Class Interpreter</H2>     International readers
may have spotted a subtle problem early on that was not handled in the listings
– numbers were being reported in the numeric format used in the United
States!   Countries like Belgium and Switzerland which use different
formats for numbers, require adjustments to the interpreter in order to work at
all.  Fortunately, the .NET framework includes built-in support for
converting numbers between different cultures, so the changes to the interpreter
required are straightforward.  In the interpreter, the only fractional
value is speed, so only one change is necessary. The NmeaCultureInfo variable
represents the culture used for numbers within NMEA sentences.  The
Double.Parse method is then used with this variable to convert speed into the
machine’s local culture.    Listing 1-8 shows the completed
interpreter, now ready for use internationally.
<P></P>
<BLOCKQUOTE dir=ltr style="MARGIN-RIGHT: 0px">
<P><EM>(Listing 1-8: The completed interpreter, suitable for use anywhere in the
world.)</EM> <PRE lang=vb.net>'*************************************************************
'** Listing 1-8. Adding support for international cultures
'*************************************************************
Imports System.Globalization
Public Class NmeaInterpreter
 ' Represents the EN-US culture, used for numers in NMEA sentences
 Private NmeaCultureInfo As New CultureInfo("en-US")
 ' Used to convert knots into miles per hour
 Private MPHPerKnot As Double = Double.Parse("1.150779", NmeaCultureInfo)
 ' Raised when the current location has changed
 Public Event PositionReceived(ByVal latitude As String, ByVal longitude As String)
 Public Event DateTimeChanged(ByVal dateTime As DateTime)
 Public Event BearingReceived(ByVal bearing As Double)
 Public Event SpeedReceived(ByVal speed As Double)
 Public Event SpeedLimitReached()
 Public Event FixObtained()
 Public Event FixLost()
 Public Event SatelliteReceived(ByVal pseudoRandomCode As Integer, _
 ByVal azimuth As Integer, _
 ByVal elevation As Integer, _
 ByVal signalToNoiseRatio As Integer)
 ' Processes information from the GPS receiver
 Public Function Parse(ByVal sentence As String) As Boolean
 ' Discard the sentence if its checksum does not match our calculated checksum
 If Not IsValid(sentence) Then Return False
 ' Look at the first word to decide where to go next
 Select Case GetWords(sentence)(0)
  Case "$GPRMC"  ' A "Recommended Minimum" sentence was found!
  Return ParseGPRMC(sentence)
  Case "$GPGSV"
  Return ParseGPGSV(sentence)
  Case Else
  ' Indicate that the sentence was not recognized
  Return False
 End Select
 End Function
 ' Divides a sentence into individual words
 Public Function GetWords(ByVal sentence As String) As String()
 Return sentence.Split(","c)
 End Function
 ' Interprets a $GPRMC message
 Public Function ParseGPRMC(ByVal sentence As String) As Boolean
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Do we have enough values to describe our location?
 If Words(3) <> "" And Words(4) <> "" _
 And Words(5) <> "" And Words(6) <> "" Then
  ' Yes. Extract latitude and longitude
  Dim Latitude As String = Words(3).Substring(0, 2) & "°"  ' Append hours
  Latitude = Latitude & Words(3).Substring(2) & """" ' Append minutes
  Latitude = Latitude & Words(4)  ' Append the hemisphere
  Dim Longitude As String = Words(5).Substring(0, 3) & "°"  ' Append hours
  Longitude = Longitude & Words(5).Substring(3) & """" ' Append minutes
  Longitude = Longitude & Words(6)  ' Append the hemisphere
  ' Notify the calling application of the change
  RaiseEvent PositionReceived(Latitude, Longitude)
 End If
 ' Do we have enough values to parse satellite-derived time?
 If Words(1) <> "" Then
  ' Yes. Extract hours, minutes, seconds and milliseconds
  Dim UtcHours As Integer = CType(Words(1).Substring(0, 2), Integer)
  Dim UtcMinutes As Integer = CType(Words(1).Substring(2, 2), Integer)
  Dim UtcSeconds As Integer = CType(Words(1).Substring(4, 2), Integer)
  Dim UtcMilliseconds As Integer
  ' Extract milliseconds if it is available
  If Words(1).Length > 7 Then
  UtcMilliseconds = CType(Words(1).Substring(7), Integer)
  End If
  ' Now build a DateTime object with all values
  Dim Today As DateTime = System.DateTime.Now.ToUniversalTime
  Dim SatelliteTime As New System.DateTime(Today.Year, Today.Month, _
  Today.Day, UtcHours, UtcMinutes, UtcSeconds, UtcMilliseconds)
  ' Notify of the new time, adjusted to the local time zone
  RaiseEvent DateTimeChanged(SatelliteTime.ToLocalTime)
 End If
 ' Do we have enough information to extract the current speed?
 If Words(7) <> "" Then
  ' Yes. Parse the speed and convert it to MPH
  Dim Speed As Double = Double.Parse(Words(7), NmeaCultureInfo) _
       * MPHPerKnot
  ' Notify of the new speed
  RaiseEvent SpeedReceived(Speed)
  ' Are we over the highway speed limit?
  If Speed > 55 Then RaiseEvent SpeedLimitReached()
 End If
 ' Do we have enough information to extract bearing?
 If Words(8) <> "" Then
  ' Indicate that the sentence was recognized
  Dim Bearing As Double = CType(Words(8), Double)
  RaiseEvent BearingReceived(Bearing)
 End If
 ' Does the device currently have a satellite fix?
 If Words(2) <> "" Then
  Select Case Words(2)
  Case "A"
   RaiseEvent FixObtained()
  Case "V"
   RaiseEvent FixLost()
  End Select
 End If
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Interprets a "Satellites in View" NMEA sentence
 Public Function ParseGPGSV(ByVal sentence As String) As Boolean
 Dim PseudoRandomCode As Integer
 Dim Azimuth As Integer
 Dim Elevation As Integer
 Dim SignalToNoiseRatio As Integer
 ' Divide the sentence into words
 Dim Words() As String = GetWords(sentence)
 ' Each sentence contains four blocks of satellite information. Read each block
 ' and report each satellite's information
 Dim Count As Integer
 For Count = 1 To 4
  ' Does the sentence have enough words to analyze?
  If (Words.Length - 1) >= (Count * 4 + 3) Then
  ' Yes. Proceed with analyzing the block. Does it contain any information?
  If Words(Count * 4) <> "" And Words(Count * 4 + 1) <> "" _
  And Words(Count * 4 + 2) <> "" And Words(Count * 4 + 3) <> "" Then
   ' Yes. Extract satellite information and report it
   PseudoRandomCode = CType(Words(Count * 4), Integer)
   Elevation = CType(Words(Count * 4 + 1), Integer)
   Azimuth = CType(Words(Count * 4 + 2), Integer)
   SignalToNoiseRatio = CType(Words(Count * 4 + 2), Integer)
   ' Notify of this satellite's information
   RaiseEvent SatelliteReceived(PseudoRandomCode, Azimuth, Elevation, _
   SignalToNoiseRatio)
  End If
  End If
 Next
 ' Indicate that the sentence was recognized
 Return True
 End Function
 ' Returns True if a sentence's checksum matches the calculated checksum
 Public Function IsValid(ByVal sentence As String) As Boolean
 ' Compare the characters after the asterisk to the calculation
 Return sentence.Substring(sentence.IndexOf("*") + 1) = GetChecksum(sentence)
 End Function
 ' Calculates the checksum for a sentence
 Public Function GetChecksum(ByVal sentence As String) As String
 ' Loop through all chars to get a checksum
 Dim Character As Char
 Dim Checksum As Integer
 For Each Character In sentence
  Select Case Character
  Case "$"c
   ' Ignore the dollar sign
  Case "*"c
   ' Stop processing before the asterisk
   Exit For
  Case Else
   ' Is this the first value for the checksum?
   If Checksum = 0 Then
   ' Yes. Set the checksum to the value
   Checksum = Convert.ToByte(Character)
   Else
   ' No. XOR the checksum with this character's value
   Checksum = Checksum Xor Convert.ToByte(Character)
   End If
  End Select
 Next
 ' Return the checksum formatted as a two-character hexadecimal
 Return Checksum.ToString("X2")
 End Function
End Class
</PRE>
<P></P></BLOCKQUOTE>
<H2>Final Thoughts</H2>
<P>     You should now have a good understanding that an
NMEA interpreter is all about extracting words from sentences.  You can
harness the power of satellites to determine your location, synchronize your
computer clock, find your direction, watch your speed, and point to a satellite
in the sky on a cloudy day.  This interpreter will also work with the .NET
Compact Framework without any modifications.  If sentences were also stored
in a file, the interpreter can be used to play back an entire road trip. 
These are all great features, especially considering the small size of the
class, but is this interpreter ready to drive your car and pilot an
airplane?  Not quite yet.  There is one important topic remaining
which is required to make GPS applications safe for the real world: 
precision.  <BR>     GPS devices are designed to report
any information they find, even if the information is inaccurate.  In fact,
information about the current location can be off as much as half a football
field, even when devices are equipped with the latest DGPS and WAAS correction
technologies!  Unfortunately, several developers are not aware of this
problem.  There are some third-party components out there which are not
suitable for commercial applications that require enforcing a minimum level of
precision.  Keep this article handy, however, because in part two of this
series, I will explain precision enforcement in detail and take the interpreter
even further to make it suitable for professional, high-precision
applications!</P>
<H2>Related Links</H2>
<UL class=Download>
<LI>http://www.planet-source-code.com/vb/scripts/ShowCode.asp?txtCodeId=3123&lngWId=10 -- Writing GPS Applications: Part 2
<LI>http://www.gpsdotnet.com/download/WritingGPSApplications1_src.zip -- Download Source Code for This Article (17K)
<LI>http://www.gpsdotnet.com/download/WritingGPSApplications2_demo_gpsdotnet.zip -- Download GPS demo using GPS.NET (373K)
<LI>http://www.gpsdotnet.com/download/setup.exe -- Download GPS.NET (Full SDK) (5,373K)
</UL>

