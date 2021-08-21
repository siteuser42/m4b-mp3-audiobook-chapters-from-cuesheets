## clunky-audiobook-chapterizing
A clunky but mostly automated way to merge audiobooks without an encoding step (remuxing only) to get either single file m4b files with quicktime/nero chapters or single file mp3s with id3v2 chapters (not widely supported unfortunately)

This is cobbled together from various scripts and software from github, googling, etc, but the majority is in .bat files so Windows is required (probably would be trivial to convert to python/ruby except for the executables)

Full credit to creators of borrowed scripts/software (TestToCue, mergechapters.py, direnhanced, ffmpeg, mp4chaps, rubycue, mp3directcut, rxrepl)

Not everything that is needed is included in this repository, will also need Ruby, python, rubycue gem and its associated gems, mp3directcut, ffmpeg (obviously), mp3tag, rxrepl, (findstr is also used but should be pre-installed on windows, found in system32 if not and can copy it into working directory)

NOTE: Every script and bat file (except ruby gems) needs to be in the working directory for this method to work correctly.

General work flow is:
1. Merge multiple files to a single file mp3/mp4 file (should be called input.mp3 or m4b to work with .bat files)
2. Obtain a .cue file (from a tracklist file, list.txt, generated by mp3tag or pause detection by mp3directcut, should be called cuesheet.cue for .bat files)
3. Covert .cue file to ffmetadata file (should be called metadata.txt for .bat files)
4. Combine ffmetadata with mp3/mp4 file (generating output.mp3 or m4b)

Any suggestions on how to improve process is appreciated

#### Working files should be called:
- input.mp3/.mp4 (this is generated by respective merge_ext.bat file)
- cuesheet.cue (this is generated by text2cue.bat or how you should name cue after saving from mp3directcut)
- metadata.txt (this is generated by cue2metadata.bat)
- output files are temporarily called output.mp3/mp4 then renamed to the album metadata field and moved to folder of same name

Temporary files are created along the way as well, but should be deleted at the end of their associated batch proccess

cleanup.bat will delete all working files (including input, excluding output) - you may want to backup your cuesheet before running this!

### Creating single M4B file with embedded chapters from individual chapterized m4b files
	Alternative method: AudioBookConverter (https://github.com/yermak/AudioBookConverter) can handle these pretty well, potentially much easier, but can be buggy at times
	0. requires:
		# ffmpeg (tested using 4.4)
		# filelistgenerator_m4b.bat
		# metadatatransfer_m4b.bat
		# merge_m4b.bat
		# mp4_to_m4b.bat + mp4chaps.exe, http://pds16.egloos.com/pds/200910/19/90/mp4_to_m4b.zip (.exe originates from MP4v2)
		# text2cue.bat
		# cue2metadata.bat
		# merge_inputmp4_with_metadata
		# rxrepl (https://sites.google.com/site/regexreplace/) - optional, checks for formatting errors but the .bat still works if it isn't present
	1. Add m4b files to bin folder of ffmpeg (4.4 plus)
	2. Import tracks into mp3tag
	3. Edit "title" fields as desired (full chapter names will take longer than simple Chapter 01) and ensure "track" fields are filled with integers (can use mp3tag > tools > auto number wizard)
	4. select all, right click > export > txt_taglist > Okay to create list.txt file
		- If this is first time, you will need to first edit "txt_taglist" and replace current text with below and save changes
			$filename(txt,utf-8)$loop(%_path%)%track%.%artist% - %title% - $div(%_length_seconds%,60)':'$num($mod(%_length_seconds%,60),2)
			$loopend()
	5. Use text2cue.bat or drag generated .txt file onto TextToCue.exe file, which generates a .cue file
		# if you manually do it, ensure the the cue has a TITLE field before next step
	6. Run merge_m4b.bat to merge m4b files to a single input.mp4 file
	7. Run cue2metadata.bat and merge_inputmp4_with_metadata.bat
	8. Run mp4tom4b.bat (converts fmpeg embedded chapters to proper m4b quicktime format), will also move output into its own folder and rename based on title field

### Creating single MP3 file with embedded chapters from individual chapterized mp3 files
	0. requires:
		# ffmpeg (tested using 4.4)
		# mp3tag (tested using 3.00d)
		# TextToCue.exe, https://community.mp3tag.de/t/generate-cue-file-from-tracklist/11750/13
		# ruby
		# rubycue
			Some edits to the cuesheet.rb file in the package are required to work for audiobooks! 
			See pending pull request to fix performer validation issues
			Then change index hours to 4 
			update: made more edits to complete turn off performer parsing validation
		# cue2ffmeta.rb
		# filelistgenerator_mp3.bat
		# metadatatransfer_mp3.bat
		# text2cue.bat
		# cue2metadata.bat
		# merge_inputmp3_with_metadata.bat
		# rxrepl (https://sites.google.com/site/regexreplace/) - optional, checks for formatting errors but the .bat still works if it isn't present
	1. Add mp3 files to bin folder of ffmpeg (4.4 plus)
	2. Import tracks into mp3tag
	3. Edit "title" fields as desired and ensure "track" fields are filled  in format with integers (i.e. 01 or 1, not "1 of 42" etc.) (can use mp3tag > tools > auto number wizard to autofill track numbers)
	4. select all, right click > export > txt_taglist > Okay to create list.txt file
		- If this is first time, you will need to first edit "txt_taglist" and replace current text with below and save changes
			$filename(txt,utf-8)$loop(%_path%)%track%.%artist% - %title% - $div(%_length_seconds%,60)':'$num($mod(%_length_seconds%,60),2)
			$loopend()
	5. Use text2cue.bat or drag generated .txt file onto TextToCue.exe file, which generates a .cue file
			# if you manually do it, ensure the the cue has a TITLE field before next step
	6. Run merge_mp3.bat to merge mp3 files to a single input.mp3 file
	7. Run cue2metadata.bat and merge_inputmp3_with_metadata.bat, will also move output into its own folder and rename based on title field
		
### Creating single MP3 file with embedded chapters from randomly split mp3 files
	0. requires:
		# ffmpeg (tested using 4.4)
		# mp3directcut
		# ruby
		# rubycue
			Some edits to the cuesheet.rb file in the package are required to work better for audiobooks! 
			See pending pull request to fix performer validation issues (currently script checks for PERFORMER field on every cue chapter entry, this removes that requirement)
			Then change index hours to 4 (required for audiobooks over 16 hours)
		# cue2ffmeta.rb
		# filelistgenerator_mp3.bat
		# metadatatransfer_mp3.bat
		# merge_mp3.bat
		# cue2metadata.bat
		# merge_inputmp3_with_metadata.bat
	1. Add mp3 files to bin folder of ffmpeg (4.4 plus)
	2. Run merge_mp3.bat to merge mp3 files to a single input.mp3 file
	3. Drag merged mp3 file into mp3directcut
	4. Go to special > pause detection
		# Try -44.5 dB, 2.9 s, 10 frames
	5. Wait for it to detect chapter breaks, click close once it no longer says "stop"
	6. Check detected chapters with the >| dotted line button
		# If undercounts chapters, repeat step 4 with lower seconds or manually find chapters to add (c creates a chapter but only if correctly selected, quickly make chapters at cursor by hitting b, n, c)
		# If it overcounts by a lot, repeat step 4 with higher seconds
		# If only a few extra, can delete by highlighting with cursor (on the upper waveform not the rectangles) and go to special > remove edit break
		# Also can cross references number of chapters with numbers of pauses using epub
	7. File > Save as cuesheet.cue
	8. Open up .cue file in text editor and change TITLES as desired for each chapter (if you haven't modified rubycue script to turn off validation, add PERFORMER "Author Name" as second line)
	9. Run cue2metadata.bat and merge_inputmp3_with_metadata.bat, will also move output into its own folder and rename based on title field
	
### Creating single M4B file with embedded chapters from randomly split m4b files
	0.5. This one is the annoying type, since mp3directcut doesn't support m4b we will have to make a temporary reencode of the file to generate the cue sheet
	1. Add m4b files to bin folder of ffmpeg (4.4 plus)
	2. Run merge_m4b.bat to merge m4b files to a single input.mp4 file
	3. Open up a cmd terminal in the current directory and type:
	> ffmpeg -i input.mp4 output.mp3
	3. Drag reencoded mp3 file into mp3directcut
	4. Go to special > pause detection
	# Try -44.5 dB, 2.9 s, 10 frames
	5. Wait for it to detect chapter breaks, click close once it no longer says "stop"
	6. Check detected chapters with the >| dotted line button
		# If undercounts chapters, repeat step 4 with lower seconds or manually find chapters to add (c creates a chapter but only if correctly selected, quickly make chapters at cursor by hitting b, n, c)
		# If it overcounts by a lot, repeat step 4 with higher seconds
		# If only a few extra, can delete by highlighting with cursor (on the upper waveform not the rectangles) and go to special > remove edit break
		# Also can cross references number of chapters with numbers of pauses using epub
	7. File > Save as cuesheet.cue
	8. Open up .cue file in text editor and change TITLES as desired for each chapter (if you haven't modified rubycue script to turn off validation, add PERFORMER "Author Name" as second line)
	9. Once satisfied with your .cue, delete reencoded output.mp4
	10. Run cue2metadata.bat and merge_inputmp4_with_metadata.bat, will also move output into its own folder and rename based on title field
	
### Creating single M4B file that retains chapers from two or more m4b files with embedded chapters
	0. requires:
		# ffmpeg (tested using 4.4)
		# python (tested using 3.9.8)
		# mergechapters.py, https://gist.github.com/cliss/53136b2c69526eeed561a5517b23cefa
	1. change .m4b to .mp4
	2. open cmd in current directory:
		> py ./mergechapters.py input1.mp4 input2.mp4 output.mp4
	3. Remove extraneous video stream if needed
		> ffmpeg -i output.mp4 -map 0 -map -0:v -c copy final.mp4
	3. change .mp4 to .m4b
	4. check with ffmpeg -i *.m4b or MediaInfo to ensure chapters are present
	
### other useful info
- quick remux can fix file errors
	> ffmpeg -i %FILENAME%.ext -c copy %FILENAME%.mp4
	> ffprobe -i input.ext -show_entries format=duration -v quiet -of csv=p=0

- old version of merge_m4b.bat allowed for custom filename entry, edited to only export as input.mp4
- merge_m4b.bat
> call direnhanced_m4b.bat > fileList.txt
> set /p FILENAME=Type Desired Output file name then hit ENTER to continue...
> ffmpeg -f concat -safe 0 -i fileList.txt -c copy "%FILENAME%.mp4"
> endlocal

- You can double check files with :
		> ffmpeg -i file.ext OR ffprobe file.ext

- To export metadata from a file, also have export.bat
> ffmpeg -i FILE.ext -f ffmetadata in.txt
or
> ffmpeg -i FILE.ext -c copy -map_metadata 0 -map_metadata:s:v 0:s:v -map_metadata:s:a 0:s:a -f ffmetadata in.txt

- remove existing chapters
> ffmpeg -i input.mp3 -map_metadata -1 -map_chapters -1 -c copy output2.mp3

### Troubleshooting Notes
- This method has trouble with special characters, often in the filename of the chapter files or occasionally in the title fields. If you are having issues, check for these characters first.
- Note that an mp3 id3v2 chapter has a character limit of 62 characters apparently (untested)
- Rubycue will have parsing issues if unedited. Comment out fields related to validation and performer to fix in cuesheet.rb file - see my fork if you want to copy it
- Rubycue by default doesn't work with audiobooks over 16 hours, change index to 4 to fix in cuesheet.rb - see my fork if you want to copy it
- Tracks in mp3tag must be integers (not fractions or phrases), TextToCue.exe is sensitive to format so double check for example no HH:MM:SS must be MM:SS (this should be automatically be taken care of now if using mp3tag)
- If Chapter 1 is missing and starts with Chapter 2 then the title field of the file took the title field of first chapter, to fix make sure you have a file TITLE field in cuesheet on first line
- Minor issue, but last chapter duration is often given as 0 if starting from tracklist (I think it is result a result of cue2ffmeta needing a following chapter to calculate end time). Can be fixed with small manual edit, duplicate the final chapter in the tracklist, then delete it in the metadata.txt file before merging with input
- ffmpeg can't handle m4b extension, but changing to mp4 allows it to work
- Had some mp3 files where adding new metadata removed the embedded chapters. This seems to occur when over 50 chapters and/or 90000000 (7 zeroes). Seems to be fixed by enforcing id3v2.3 tags. If still having issues, may have to fix by combining the extra chapters or splitting into two files. If anyone has other solutions please share.