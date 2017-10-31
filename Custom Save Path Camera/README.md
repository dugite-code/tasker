#Take Photo with custom save path shortcut

[Reddit Post](https://www.reddit.com/r/tasker/comments/79sj7y/how_to_take_photo_with_custom_save_path_shortcut/)
Thanks to /u/kholdstayr for solving the camera intent issue I was having in my [previous Help post](https://www.reddit.com/r/tasker/comments/786dk5/help_camera_intent_custom_save_path/).

This is broken down into 2 Tasks. 1 a simple shortcut task passes the save path and the date format and the other issues the Camera intent and then the media scanner intent so the new photo shows in your Gallery apps.


#Camera Task

Fetch your device specific SDCard path

    A1: Java Function [ Return:%folderpath Class Or Object:Environment Function:getExternalStorageDirectory

      {File} () Param: Param: Param: Param: Param: Param: Param: ]

Add the relative save path submitted through %par1 to the absolute SDCard path

    A2: Variable Set [ Name:%folderpath To:%par1 Recurse Variables:Off Do Maths:Off Append:On ]

Fetch the current date

    A3: Java Function [ Return:date Class Or Object:Date Function:new

      {Date} () Param: Param: Param: Param: Param: Param: Param: ]

Format the date using string submitted through %par2

    A4: Java Function [ Return:%timestamp Class Or Object:android.text.format.DateFormat Function:format

      {CharSequence} (CharSequence, Date) Param:"%par2" Param:date Param: Param: Param: Param: Param: ]

Build absolute file path. **Note:** I added *IMG_* before the timestamp so my pictures would match the OpenCamera naming format. You can change it here

    A5: Variable Set [ Name:%filepath To:%folderpath/IMG_%timestamp Recurse Variables:Off Do Maths:Off Append:On ]

Add file extension to file path

    A6: Variable Set [ Name:%filepath To:.jpg Recurse Variables:Off Do Maths:Off Append:On ]

Declare a new file object

    A7: Java Function [ Return:file Class Or Object:File Function:new

      {File} (String) Param:%filepath Param: Param: Param: Param: Param: Param: ]

Convert file object into URI object

    A8: Java Function [ Return:uri Class Or Object:Uri Function:fromFile

      {Uri} (File) Param:file Param: Param: Param: Param: Param: Param: ]

Convert file object into URI object but this time as a tasker variable string

    A9: Java Function [ Return:%uri Class Or Object:Uri Function:fromFile

      {Uri} (File) Param:file Param: Param: Param: Param: Param: Param: ]

Issue the camera intent. Note as /u/kholdstayr pointed out the URI object must be used and not the tasker variable.

    A10: Send Intent [ Action:android.media.action.IMAGE_CAPTURE Cat:None Mime Type: Data: Extra:output:uri Extra: Extra: Package: Class: Target:Activity ]

Loop until the new file exists

    A11: Test File [ Type:Exists Data:%filepath Store Result In:%fileexists Use Root:Off ]

    A12: Wait [ MS:0 Seconds:1 Minutes:0 Hours:0 Days:0 ]

    A13: Goto [ Type:Action Number Number:11 Label: ] If [ %fileexists neq true ]

Issue the Media Scanner intent letting the media scanner know about our new file

    A14: Send Intent [ Action:android.intent.action.MEDIA_SCANNER_SCAN_FILE Cat:None Mime Type: Data:%uri Extra: Extra: Extra: Package: Class: Target:Broadcast Receiver ]

A reminder as to where the file is stored

    A15: Flash [ Text:Saved to %filepath Long:Off ]

***

#Shortcut Task

**Note:** the path must not end in a / or the path will build as /storage/emulated/0/Pictures/Receipts//IMG_

[See this link](https://www.unicode.org/reports/tr35/tr35-dates.html#Date_Format_Patterns) for date formats. I have it as yyyyMMdd_kkmmss. So it will show year,month,day and _ then Hours,minutes,seconds i.e. 20171031_015523

    A1: Perform Task [ Name:Camera Priority:50 Parameter 1 (%par1):/Pictures/Receipts Parameter 2 (%par2):yyyyMMdd_kkmmss Return Value Variable:%uri Stop:Off ]