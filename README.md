## m4b-mp3-audiobook-chapters-from-cuesheets
- Mostly automated way to merge audiobooks without quality loss from reencoding (remuxing only) to either single file m4b files with quicktime/nero chapters or single file mp3s with id3v2 chapters using cuesheets
	- [Good breakdown of different chapter types](https://github.com/Zeugma440/atldotnet/wiki/Focus-on-Chapter-metadata)

- Windows is required (majority is written in batch files along with executables)

#### Full credit to creators of borrowed scripts/software (included in repository for convenience will remove if asked):
- [cue2ffmeta.rb](https://gist.github.com/remko/e15c4fe26d479e134f36#file-cue2ffmeta-rb)
- [TextToCue.exe](https://community.mp3tag.de/t/generate-cue-file-from-tracklist/11750/13)
- [mp4chaps](https://code.google.com/archive/p/mp4v2/) (originates from MP4v2)
- [mergechapters.py](https://gist.github.com/cliss/53136b2c69526eeed561a5517b23cefa) (small edit made to remove extraneous video streams from embedded covers)

#### Not everything that is required is included in this repository! You will also need:
- [ffmpeg.exe & ffprobe.exe](https://www.gyan.dev/ffmpeg/builds/) (tested with 4.4)
- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [rubycue](https://rubygems.org/gems/rubycue/versions/0.1.0) 
	- An edited cuesheet.rb is provided, which allows for 16+ hour books and turns off validation parsing, it should be placed in your-ruby-install-location\Ruby30-x64\lib\ruby\gems\3.0.0\gems\rubycue-0.1.0\lib\rubycue"
- [mp3directcut](https://mpesch3.de/) (v1.99 tested)
- [mp3tag](https://www.mp3tag.de/en/download.html)
- [python3](https://www.python.org/downloads/) (optional, only for Scenario 5)

#### General work flow is:
1. Merge multiple audio files to a single mp3/mp4 file
2. Obtain and edit cuesheet to add desired chapter info
	- generated by tracklist (from mp3tag) or pause detection (by mp3directcut)
3. Convert .cue to ffmetadata then combine ffmetadata with mp3/mp4 file

### Installation
- Install the above required programs separately
- Place all the .bat files and cue2ffmeta.rb and mergechapters_ab.py in the bin folder of ffmpeg (this is your working directory)
- Ensure cuesheet.rb in your rubycue installation was replaced with the copy provided
- Go to the desired scenario below and follow its steps

#### Working files should be called:
- input.mp3/.mp4 (from merging individual chapters or existing single file)
- cuesheet.cue (this is generated by respective text2cue.bat or from mp3directcut)
- output.mp3/mp4, will auto attempt to rename with album metadata field and move to folder of same name
	- note if album field is blank it will be renamed back to output
- Temporary files are created as well, but should be deleted at the end of their associated batch process

- PLEASE NOTE: Special characters can cause issues in several steps. I've added some support for the most common, but still an issue with many, especially language characters

### Scenario 1: Creating single M4B file with embedded chapters from individual chapterized m4b files
	1. Add m4b files to bin folder of ffmpeg (4.4 plus)
	2. Import tracks into mp3tag
	3. Edit "title" fields as desired (full chapter names will take longer than simple Chapter 01)
		- Ensure Tracknumber fields are filled with integers (can use mp3tag > tools > auto number wizard) 
		- Ensure the Artist field is not empty
		- Ensure the Album field contains no characters not allowed in a filename (?, :, etc)
	4. Select all, right click > export > txt_taglist > Okay to create list.txt file
		- If this is first time, you will need to first edit "txt_taglist" and replace current text with below and save changes
			$filename(txt,utf-8)$loop(%_path%)%track%.%artist% - %title% - $div(%_length_seconds%,60)':'$num($mod(%_length_seconds%,60),2)
			$loopend()
	5. Use "m4b_text2cue.bat"
		- will generate a .cue file
	6. Run m4b_merge-files.bat
		- will merge m4b files to a single input.mp4 file (make sure tracks are in proper order by filename)
	7. Run m4b_add-chapters.bat 
		- will add chapters then convert fmpeg embedded chapters to proper m4b quicktime format
		- this will also attempt to rename and move output into its own folder based on album field

### Scenario 2: Creating single MP3 file with embedded chapters from individual chapterized mp3 files
	1. Add mp3 files to bin folder of ffmpeg (4.4 plus)
	2. Import tracks into mp3tag
	3. Edit "title" fields as desired (full chapter names will take longer than simple Chapter 01)
		- Ensure Tracknumber fields are filled with integers (can use mp3tag > tools > auto number wizard) 
		- Ensure the Artist field is not empty
		- Ensure the Album field contains no characters not allowed in a filename (?, :, etc)
	4. Select all, right click > export > txt_taglist > Okay to create list.txt file
		- If this is first time, you will need to first edit "txt_taglist" and replace current text with below and save changes
			$filename(txt,utf-8)$loop(%_path%)%track%.%artist% - %title% - $div(%_length_seconds%,60)':'$num($mod(%_length_seconds%,60),2)
			$loopend()
	5. Use "mp3_text2cue.bat"
		- will generate a .cue file
	6. Run mp3_merge-files.bat
		- will merge mp3 files to a single input.mp3 file (ensure tracks are in proper order by filename)
	7. Run mp3_add-chapters.bat 
		- will add chapters in id3v2.3 format
		- this will also attempt to rename and move output into its own folder based on album field
		
### Scenario 3: Creating single MP3 file with embedded chapters from randomly split mp3 files
	1. Add mp3 files to bin folder of ffmpeg (4.4 plus)
	2. Run mp3_merge-files.bat
		- will merge mp3 files to a single input.mp3 file (ensure tracks are in proper order by filename)
	3. Drag merged input.mp3 into mp3directcut
	4. Go to special > pause detection, try -44.5 dB, 2.9 s, -6 frames as a starting point
	5. Wait for it to detect chapter breaks, click close once it no longer says "stop"
	6. Check detected chapters with the >| dotted line button
		- If undercounts chapters, repeat step 4 with lower seconds or manually find chapters to add 
			- Note c creates a chapter but only if start and end point of selection are the same
			- this can be done quickly at position by hitting b, n, c
		- If it overcounts by a lot, repeat step 4 with higher seconds
		- Incorrect chapters can be deleted by selecting section, special > remove edit break
		- It is helpful to cross references number of chapters with numbers of pauses using epub
	7. File > Save as cuesheet.cue
	8. Open up .cue file in text editor and change TITLES as desired for each chapter 
	9. Run mp3_add-chapters.bat 
		- will add chapters in id3v2.3 format
		- this will also attempt to rename and move output into its own folder based on album field
	
### Scenario 4: Creating single M4B file with embedded chapters from randomly split m4b files
	0. Since mp3directcut doesn't support m4b we will make a *temporary* reencode to generate the .cue
	1. Add m4b files to bin folder of ffmpeg (4.4 plus)
	2. Run m4b_merge-files.bat
		- will merge m4b files to a single input.mp4 file (ensure tracks are in proper order by filename)
	3. Open up a cmd terminal in the current directory and type:
	> ffmpeg -i input.mp4 output.mp3
	3. Drag reencoded mp3 file into mp3directcut
	4. Go to special > pause detection, Try -44.5 dB, 2.9 s, -6 frames as starting point
	5. Wait for it to detect chapter breaks, click close once it no longer says "stop"
	6. Check detected chapters with the >| dotted line button
		- If undercounts chapters, repeat step 4 with lower seconds or manually find chapters to add 
			- Note c creates a chapter but only if start and end point of selection are the same
			- this can be done quickly at position by hitting b, n, c
		- If it overcounts by a lot, repeat step 4 with higher seconds
		- Incorrect chapters can be deleted by selecting section, special > remove edit break
		- It is helpful to cross references number of chapters with numbers of pauses using epub
	7. File > Save as cuesheet.cue
	8. Open up .cue file in text editor and change TITLES as desired for each chapter 
	9. Once satisfied with your .cue, delete reencoded output.mp4
	9. Run mp4_add-chapters.bat 
		- will add chapters then convert fmpeg embedded chapters to proper m4b quicktime format
		- this will also attempt to rename and move output into its own folder based on album field
	
### Scenario 5: Creating single M4B file that retains chapers from two m4b files with embedded chapters (must repeat if if >2)
	1. change the names of the two m4b files to be merged to input1.mp4 and input2.mp4
	2. Run m4b_merge-chapterized.bat, which will merge the two files
		- will combine chapters then convert fmpeg embedded chapters to proper m4b quicktime format
		- will also remove extraneous video streams caused by embedded covers
	3. Can check with ffmpeg -i *.m4b or MediaInfo to ensure chapters are present
	4. If there is more than two m4b files, repeat steps for each additional part

### Scenario 6: Changing Existing Chapter Information in M4B or MP3
	1. Add the file to bin folder of ffmpeg (4.4 plus)
	2. Rename the file to export.mp3 or export.m4b
	3. Run mp3_export-metadata.bat or m4b_export-metadata.bat, respectively
	4. Open metadata.txt and edit the chapter titles as desired
	5. Rename the file to input.mp3 or input.mp4, respectively
	9. Run mp3_mergemetadata.bat or mp4_mergemetadata.bat 
		- will overwrite existing chapters with edited chapters
		- this will also attempt to rename and move output into its own folder based on album field
		
### Tips & Tricks
- These files were meant to be used with Prologue iOS app, check it out if you haven't

- When editing cuesheets, I recommend recording macros on Notepad++ to quickly edit chapter titles (use regular expressions to replace Track XX with Chapter XX)
- Also, you can copy and paste chapter titles from Calibre's Table Of Contents viewer if you have ebook

- renamefiles.bat is just a lazy way to rename any .mp3/mp4 and .cue (should only have one each) to input.mp3/mp4 and cuesheet.cue

- You can double check chapters with:
> ffmpeg -i file.ext OR ffprobe file.ext

- To remove existing chapters from a file:
> ffmpeg -i input.mp3 -map_metadata -1 -map_chapters -1 -c copy output2.mp3

### Troubleshooting Notes
- This method has trouble with special characters, often in the filename of the chapter files or occasionally in the title fields. If you are having issues, check for these characters first.
- Note that an mp3 id3v2 chapter has a character limit of 62 characters apparently (untested)
- If having performer valdiation issues, make sure cuesheet.rb was replaced (see above)
- If cue is malformed and the audiobook is over 16 hours, make sure cuesheet.rb was replaced; otherwise cue may be improperly formatted
- Tracks in mp3tag must be integers (not fractions or phrases)
- TextToCue.exe is sensitive to format, for example HH:MM:SS must be MM:SS (this should be automatically formatted correctly if using provided export options in mp3tag)
- If Chapter 1 is missing and starts with Chapter 2 then the title field of the file took the title field of first chapter, to fix make sure you have a file TITLE field in cuesheet on first line
- ffmpeg can't handle m4b extension, but changing to mp4 allows it to work
- Had some mp3 files where adding new metadata removed the embedded chapters. This occurred when over 50 chapters and/or 9e7 milliseconds. Apparently was fixed by enforcing id3v2.3 tags, but if issues persist then may have to combine the excess chapters or split into two files.