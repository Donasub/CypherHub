# CypherHub
A small browser-based cipher for trading messages with a friend who shares a keyword. Pairs a Vigenère shift with a zigzag rotation through binary, decimal, and character formats. No backend, nothing leaves your tab

What it does
You type a message and a keyword. The app produces ciphertext like this:
01100100 | 101 | g | p | 107
Anyone with the same keyword can paste that into the decode panel and recover the original message. Anyone without it sees the format pattern but recovers shifted gibberish.
The cipher has two layers:

Vigenère shift each letter is bumped forward in the alphabet by an amount taken from the keyword. The keyword cycles for messages longer than it is.
Zigzag formatting each character is then rendered in one of three formats (8-bit binary, decimal ASCII, or the character itself), rotating through them in a bouncing pattern that repeats every six positions.

Neither piece is hard on its own. Stacked, they're enough for puzzles, low-stakes notes, and party trivia. They are not enough for protecting anything important see the security caveat below.
Run it
Open zigzag-cipher.html in any modern browser. Nothing to install. No build step. No backend.

To share send someone the URL or a copy of the file. The page loads with a working hello / wave example so they can see the cipher running immediately.
To self-host serve the file from any static host (GitHub Pages, Netlify, S3, python -m http.server).
To verify open the browser console. You should see Zigzag self-tests: all 5 pass.

The cipher
Three character formats
Every printable ASCII character has three valid representations:
FormatDescriptionExample: a (ASCII 97)B · Binary8-bit ASCII in base 201100001D · DecimalASCII code in base 1097C · Characterthe character itselfa
The zigzag pattern
The format used at position i (1-indexed) depends only on the position, not on the character. The pattern bounces:
position : 1 2 3 4 5 6 7 8 9 10 11 12
format   : B D C C D B B D C C  D  B
Forward through B → D → C, bounce, backward through C → D → B, bounce again. Period 6.
jsformatAt(i) = ['B','D','C','C','D','B'][(i - 1) % 6]
Two modes
Basic mode (no keyword). Each character is rendered in its position's format. Anyone who knows the rule can read it — useful for teaching, not for hiding anything. The interface shows a small "no key — anyone can read this" badge when this mode is active.
Strong mode (with keyword). Encoding at position i:

Take the keyword character at position ((i - 1) mod L) + 1, where L is the cleaned keyword length.
Its alphabet index (a = 0, b = 1, …, z = 25) is the shift amount.
Move the plaintext letter forward in the alphabet by that amount, wrapping past z back to a. Case is preserved. Non-letters (digits, punctuation, spaces) pass through unchanged.
Apply the zigzag format for position i to the shifted character.

Decoding runs the same steps in reverse.
Worked example
Encoding hello with key wave:
posplainkeyshiftshiftedformattoken1hw22dB011001002ea0eD1013lv21gCg4le4pCp5ow22kD107
Ciphertext: 01100100 | 101 | g | p | 107
Without the key, a decoder would recover degpk — the shifted intermediate, not the original hello. This intermediate is shown explicitly in both panels so the keyword's effect is visible rather than hidden inside the encoding step.
Inside the app
Four panels, top to bottom:

Shared keyword. The input both sender and receiver must agree on. Below it, a small strip shows the per-position shifts (w +22 · a +0 · v +21 · e +4).
Encode. Plaintext on top → shifted intermediate in the middle → token output below. A copy button puts canonical ciphertext on your clipboard.
The zigzag. An inline SVG showing the format-at-each-position pattern across twelve positions, drawn as the bouncing path the cipher is named after.
Decode. Ciphertext on top → editable after-shift line in the middle → recovered plaintext below. Either field drives the output: paste tokens into the top, or paste shifted text into the middle and decode straight from there.

The "How this works" section at the top of the page walks through the same three concepts in plain language.
A theme button (top right) cycles auto → light → dark. Auto follows your system preference.
Test cases
Encode-side reference vectors:
bad     (no key)   →   01100010 | 97 | d
bade    (no key)   →   01100010 | 97 | d | e
dbade   (no key)   →   01100100 | 98 | a | d | 101
hello   "wave"     →   01100100 | 101 | g | p | 107
Roundtrip property: for any plaintext X and any keyword K,
decode(encode(X, K), K) === X
This holds for messages with spaces, punctuation, mixed case, and digits. The app verifies a representative set of these on every page load and logs the result to the browser console.
Privacy

Local only. Plaintext, keywords, and ciphertext never reach a server. There is no server.
No telemetry on input. Nothing you type in the boxes leaves your machine.
No storage. Nothing is persisted between sessions. Refresh the page and the boxes reset to the example.
Network requests at page load are limited to two font files from Google Fonts. After that, the page is fully offline-capable.

Security
This is puzzle-grade encryption. Worth understanding before you trust it with anything:

Short keys fall to Kasiski analysis. Once an attacker guesses the keyword length, the cipher collapses into a sequence of Caesar shifts solvable with letter-frequency analysis.
One-time-pad strength requires a key as long as the message, generated randomly, and never reused. This app doesn't do that.
Key distribution is unsolved here. You and your friend have to share the keyword some other way before exchanging messages. Real cryptography (RSA, Diffie–Hellman, modern TLS) is built around solving exactly that problem.
The format pattern is not a secret. Anyone who finds this app can see the B D C C D B rotation. Security rests entirely on the keyword.

Use Zigzag for notes, puzzles, party games, and trivia. Don't use it for anything you'd be upset to see leak.
Files

zigzag-cipher.html — the whole application in one file. HTML, CSS, JavaScript, and the cipher logic. No build step. No dependencies beyond two web fonts.

License
MIT license declared. Treat as personal/educational use unless you add one.
