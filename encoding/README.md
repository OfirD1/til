# Encoding  

## Misinterpreting similar encodings  

### 1. Description 
You open a text file with some text editor, and some characters look weird:   
`abcd`  

Here's the same text in Notepad++, where the character is rendered as `SPA`:

![spa](https://user-images.githubusercontent.com/30159171/80911567-c2455080-8d3f-11ea-8e5b-1c3f1804c6a1.png)

<sup>*Note that older Notepad++ versions doesn't display the "wrong" character*</sup> 

How did this `SPA` character get there?

Generally speaking, what probably happened is that this file was originally created using one encoding, but then was 
misinterpreted as if it was created using another encoding *that shares the same character-set mapping for most 
(but not all) values*. 

To be more specific, let's take 2 examples:

1. Misinterpreting a file encoded in Windows-1252 as if it's ISO-8859-1 (both encodes Latin character set).
2. Misinterpreting a file encoded in Windows-1255 as if it's ISO-8859-8 (both encodes Hebrew character set).

In both examples, byte hex value `0x96` encodes a different character:  
While in Windows 1252 (or Windows 1255) it encodes a dash 
([en dash](https://en.wikipedia.org/wiki/Dash#Common_dashes_and_Unicode_characters), to be specific), 
in ISO-8859-1 (or ISO-8859-8) it encodes a [control character](https://en.wikipedia.org/wiki/C0_and_C1_control_codes?oldformat=true) 
known as `SPA`.

- ***Side note:***  
  ***ISO/IEC 8859-1 VS. ISO-8859-1 (also: ISO/IEC 8859-8 VS. ISO-8859-8)***   
    
  You won't find the control characters on the Wikipedia article for [ISO/IEC 8859-1](https://en.wikipedia.org/wiki/ISO/IEC_8859-1?oldformat=true), because: 
  > ISO/IEC 8859-1 (the standard) only specifies the characters for the 20-7E and A0-FF byte ranges.  
The characters for bytes 00-1F and 7F-9F are left undefined.  
The ISO-8859-1 character map registered with the IANA fills the missing spots with the C0 and C1 control sets (defined elsewhere), 
thus covering 00-FF.  

  Put another way:
  > ISO-8859-1 is the IANA preferred name for this standard when supplemented with the C0 and C1 control codes from ISO/IEC 6429.  

See also [here](https://unix.stackexchange.com/questions/495643/strange-character-in-a-file).  

### 2. Reproducing
Let's reproduce it (using C#): 

<sup>if you're on .Net Core, there's [another line to add](https://stackoverflow.com/a/37870346/3002584))</sup>

```
byte[] latin = Array.ConvertAll(new int[] { 97, 98, 150, 99, 100 }, Convert.ToByte);
string latin_windows_1252 = Encoding.GetEncoding("windows-1252").GetString(latin);
string latin_ISO_8859_1 = Encoding.GetEncoding("iso-8859-1").GetString(latin);
Console.WriteLine(latin_windows_1252);              // outputs: ab-cd
Console.WriteLine(latin_ISO_8859_1);                // outputs: abcd
File.WriteAllText("latin.txt", latin_ISO_8859_1);   // save as a .txt file. 
// go ahead and open the file with Notepad++ to observe the `SPA` character
```

Now, usually, you wouldn't be the one who created this file, and therefore you wouldn't work directly with either of those encodings.  
Instead, you'd get the file from whatever source as a Unicode encoded file.  
To imitate this common case, let's *transcode* the original bytes to UTF-8:
```
byte[] latin_transcoded = Encoding.Convert(Encoding.GetEncoding("windows-1252"), Encoding.UTF8, latin);
string latin_utf8 = Encoding.UTF8.GetString(latin_transcoded);
Console.WriteLine(latin_utf8); // outputs: abcd
```

Of course, after transcoding, the `SPA` character is still there.  

How can we remove it? we have 2 options:  
1. Replace the `SPA` with a dash ([see here](https://stackoverflow.com/a/56579565/3002584)).
2. Just remove every control character ([see here](https://stackoverflow.com/a/39774963/3002584)).

---------------

## Working with encoding in MSSQL 

### 1. Default encodings ([source](https://stackoverflow.com/questions/5182164/sql-server-default-character-encoding))
Unicode data (i.e. that which is found in `N`-prefixed and `XML` types) is stored in UCS-2\UTF-16 
(storage is the same, UTF-16 merely handles Supplementary Characters correctly).  

Non-Unicode data (i.e. that which is found in the `CHAR`, `VARCHAR`, and `TEXT` types — but don't use `TEXT`, use `VARCHAR(MAX)` instead)
uses an 8-bit encoding (Extended ASCII, DBCS, or EBCDIC).  
The specific encoding is based on the Code Page, which in turn is based on the Collation of a column, 
or the Collation of the current database, 
or the Collation of the Instance, 
or what is specified in a `COLLATE` clause if one is being used.  
SQL Server 2019 introduces native support for UTF-8 in `VARCHAR` and `CHAR` datatypes (not `TEXT`).

### 2. Getting underlying bytes  
To get the hex byte representaion of a field, run:  
```
SELECT convert(varbinary(max), field_name), field_name
FROM table_name
```
