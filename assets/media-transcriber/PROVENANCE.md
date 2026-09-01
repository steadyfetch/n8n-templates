# media-transcriber sample clip — provenance

`gettysburg-address-librivox-25s.mp3` — the first 25 seconds of the LibriVox recording of
Abraham Lincoln's Gettysburg Address (reader's LibriVox intro + "Four score and seven years ago…").

- Source: https://archive.org/details/gettysburg_address_librivox (file `gettysburg_address_lincoln_64kb.mp3`)
- Licence: public domain (LibriVox recordings are released into the public domain; archive.org
  `licenseurl` = creativecommons.org/licenses/publicdomain/). The text is public domain (1863).
- Processing: `ffmpeg -t 25 -ac 1 -ar 44100 -b:a 64k` on 2026-09-02; 25.05 s, 201 KB, md5 `9137c8ba7f9064a7e4ed7ffb28998e17`.
- Use: the fixed default sample for the steadyfetch **Speech to Text / media-transcriber** actor — a
  bare Start with no URLs transcribes this clip (charged like any run, one audio minute). Hosted on
  our own repo so the sample never depends on a third-party URL that can rot.
