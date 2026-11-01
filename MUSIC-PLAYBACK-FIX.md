# Music discovery correction

Release: **2026.10.31-dashboard.4**

The dashboard already selects `Music/PR_1.mp3` and `Music/PR_2.mp3`, but it first checks for available tracks with HTTP **HEAD** requests. The supplied local bridge previously supported GET only, so those discovery checks returned HTTP 501 and the playlist could remain empty. Earlier installation checks verified GET/file delivery, not this HEAD path or audible speaker output.

This release adds HEAD support with the same origin and file-path protections as GET. Successful HEAD responses provide content length/type without reading or sending the MP3 body. The player, tracks and volume behaviour are unchanged.

The updated release-install check probes all 20 track names exactly as the dashboard does and requires discovery of **PR_1 and PR_2**. It also checks their audio content type and file size. The two supplied MP3s are checked for full decoding with FFmpeg, and a separate JavaScript player test checks starting either track, switching at track end and retrying playback after a user gesture.

Upload the new matching `package.zip` and `update.json` (JSON last or both in one commit). Reboot after the update so the previous kiosk/HOME changes and this bridge correction all run together. All prior NFC, HOME, icons, workout app and music files remain included.

**On-Pi confirmation still required:** listen for music through the selected speaker/HDMI output. If autoplay is blocked, press a remote key once. Passing software/decoding checks does not establish the Pi's speaker selection, hardware volume or audible output.
