Put her daily photos in this folder.

Suggested naming: day-01.jpg, day-02.jpg, day-03.jpg ... day-40.jpg

Recommended size: 1200-1600 px on the long edge, ~200-400 KB per photo
(bigger photos make the map slow to load). You can resize in bulk with
any photo app or with a one-liner:

  # macOS / Linux (requires ImageMagick)
  mogrify -resize 1400x1400\> -quality 82 *.jpg

Then reference the filename in data/journal.json like:

  "photo": "photos/day-07.jpg"
