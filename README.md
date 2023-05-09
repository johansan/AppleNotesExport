# AppleNotesExport
AppleScript to export Apple Notes into folders as HTML

```
-- Apple Notes Exporter version 1.0
-- By Johan Sanneblad
-- Exports all Apple Notes in folders with filename "<creation date> <title> [<id>]"
set exportFolder to (choose folder) as string

-- Simple text replacing
on replaceText(find, replace, subject)
	set prevTIDs to text item delimiters of AppleScript
	set text item delimiters of AppleScript to find
	set subject to text items of subject
	
	set text item delimiters of AppleScript to replace
	set subject to "" & subject
	set text item delimiters of AppleScript to prevTIDs
	
	return subject
end replaceText

-- Returns an HTML file to save the note in.  We have to escape
-- the colons or AppleScript gets upset.
-- noteContainer must end with ":"
on noteNameToFilePath(noteContainer, noteName)
	global exportFolder
	set strLength to the length of noteName
	
	if strLength > 250 then
		set noteName to text 1 thru 250 of noteName
	end if
	
	set noteFolder to exportFolder & noteContainer
	
	-- Create folder if it does not exist
	set outputFolder to POSIX path of (noteFolder as text)
	do shell script "mkdir -p " & quoted form of outputFolder
	
	set fileName to (noteFolder & replaceText(":", "_", noteName) & ".html")
	return fileName
end noteNameToFilePath


tell application "Notes"
	
	repeat with theNote in notes of default account
		
		set noteLocked to password protected of theNote as boolean
		set noteModificationDate to modification date of theNote as date
		set noteCreationDate to creation date of theNote as date
		
		set noteID to id of theNote as string
		set oldDelimiters to AppleScript's text item delimiters
		set AppleScript's text item delimiters to "/"
		set theArray to every text item of noteID
		set AppleScript's text item delimiters to oldDelimiters
		set noteFolder to ""
		
		if length of theArray > 4 then
			-- the last part of the string should contain the ID
			-- e.g. x-coredata://39376962-AA58-4676-9F0E-6376C665FDB6/ICNote/p599
			set noteID to item 5 of theArray
		else
			set noteID to ""
		end if
		
		if not noteLocked then
			
			set theText to body of theNote as string
			set theContainer to container of theNote
			
			try
				repeat while theContainer is not missing value
					set noteFolder to (name of theContainer) & ":" & noteFolder
					set theContainer to (container of theContainer)
				end repeat
			end try
			
			-- prefix all notes by creation date (YYY-MM-DD)
			set yy to (year of noteCreationDate as text)
			-- set yy to text -2 through -1 of (year of noteCreationDate as text)
			set mm to text -2 through -1 of ("0" & (month of noteCreationDate as integer))
			set dd to text -2 through -1 of ("0" & (day of noteCreationDate))
			set datePrefix to yy & mm & dd
			
			-- file name composed by creation date + note title + id (to prevent overwriting files from the same date with same title)
			set fileName to ((datePrefix & " " & name of theNote as string) & " [" & noteID & "]") as string
			set filepath to noteNameToFilePath(noteFolder, fileName) of me
			set noteFile to open for access filepath with write permission
			
			write theText to noteFile as «class utf8» -- if you want UTF-16 replace this with "as Unicode text"
			
			close access noteFile
			
			tell application "Finder"
				set modification date of file (filepath) to noteModificationDate
			end tell
		end if
		
	end repeat
	
end tell
```
