## NDro8 Puzzle Encoding Specification - Version 1.0

This document defines the textual encoding for various logic puzzles using a JSON string format, intended to be read by supporting software for visual presentation and user interaction on mobile and desktop devices.

### 1. Core JSON Structure

All NDro8 encoded puzzles adhere to a standard JSON string format with the following attributes:

```json
{
  "spec": "ndro8",
  "ver": "1",
  "pzname": "Puzzle Name : Source and Date",
  "pztype": "puzzle_type",
  "pzlayout": "Encoded Puzzle Layout String",
  "pzdata": "Additional Puzzle Data (if required)",
  "credit": "Short Copyright String"
}
```

**Attribute Definitions:**

* **`spec`** (String, required):
    * Value: `"ndro8"`
    * Purpose: Identifies the specification used for encoding the puzzle.

* **`ver`** (String, required):
    * Value: `"1"`
    * Purpose: Indicates the version of the NDro8 specification used.

* **`pzname`** (String, required):
    * Purpose: A human-readable name for the puzzle. It can include the puzzle's specific title (if any), its source (e.g., "Times of India"), and the date of publication or creation.
    * Example: `"Hitori : Times of India 20/05/2023"`

* **`pztype`** (String, required):
    * Purpose: Defines the type of puzzle being encoded. This attribute dictates how the `pzlayout` and `pzdata` attributes are interpreted.
    * Supported values in Version 1.0:
        * `"sudoku"`
        * `"kakuro"`
        * `"hitori"`
        * `"loop"`
        * `"hidato"`

* **`pzlayout`** (String, required):
    * Purpose: This is the core encoding of the puzzle grid or structure. The format of this string is highly dependent on the `pztype` and will be defined in detail for each puzzle type.
    * First 4 Characters: The first 2 characters in pzlayout represents the width of the puzzle and next 2 characters represent height of the puzzle. Unit of width and height is 'Square', i.e. number of squares placed side by side along the width, or number of squares placed one below the other along the height. Sudoku, for example, generally are 9x9 square puzzles and thus would start with "0909".
    * Rest of the string varies for each type of puzzle, so refer specs for individual type.

* **`pzdata`** (String, optional):
    * Purpose: This attribute carries additional data specific to certain puzzle types that cannot be fully captured by `pzlayout`. Its content and format will be defined per `pztype`. If not required for a specific puzzle type (like Loop & Hitori), it should be an empty string `""`.

* **`credit`** (String, optional):
    * Purpose: A short copyright or attribution string for the puzzle's creator or source.
    * Example: `"© 2023 Puzzle Creator"` or `"© The Times of India"`

### 2. Puzzle Type Specific Definitions

This section details the encoding rules for `pzlayout` and `pzdata` for each supported `pztype`.

#### 2.1. Sudoku (`pztype`: "sudoku")

Sudoku puzzles are typically 9x9 grids where each cell eaither contains a digit from 1-9 or is empty.

* **`pzlayout`**:
    * Format: A string of 85 characters, first 4 representing width and height as described earlier. Next 81 characters represent the region in 9x9 grid. Why is it here? Well, Sudoku's got other varients like Jigsaw Sudoku & Killer Sudoku that will be specified in next version of this document and those would need the region declaration. As for Classic Sudoku there would be 9 regions, each a square of 3x3. When each region is numbered 1 to 9 the cells occupying the region will be as in example below.
    * Example for a Classic 9x9 Sudoku:
        `"0909111222333111222333111222333444555666444555666444555666777888999777888999777888999"`

* **`pzdata`**: 
    * Format: A string of 81 characters representing the 9x9 grid, read row by row, from left to right.
    * Each character represents a cell:
        * `'1'` to `'9'`: Represents a pre-filled digit in the cell.
        * `' '` i.e. a space: Represents an empty cell.
    * Example for a standard 9x9 Sudoku:
        `"  3 2 6  9  3 5  1  18 64    81 29  7       8  67 82    26 95  8  2 3  9  5 1 3  "`

#### 2.2. Kakuro (`pztype`: "kakuro")

Kakuro puzzles involve a grid with "clue" cells and "answer" cells. Clue cells contain sums for contiguous blocks of answer cells. The Kakuro puzzles can be of variable width and height with no defined standard size. If it has a width of 'w' squares and height of 'h' squares the details will be as below:

* **`pzlayout`**:
    * Format: A string of 4 + ( w x h ) characters, first 4 representing width and height as described earlier. Next set of characters represent each cell as either '1' or '0'. '1' when the cell is an answer cell to be filled by player. '0' if the cell is a 'dark' cell either blank or containing 'clues', and is not meant to be filled by player.
    * Example for a 10x10 Kakuro appearing in Times of India may be defined as below:
        `"10100000000000011011011001101101110011100011000110111001110111000110111000001110110000011101100000110110"`

* **`pzdata`**: 
    * Format: A string of characters. The number of characters will depend on how many clues are present in the puzzle. If their number is 'n' the length of this string would be '2xn'. Clues greater than 9 will be written as-is and a single digit clue needs to be padded with 0 on left.
    * The clues need to be read row by row from left to right. If a cell contains two clues one for vertical sums and other for horizontal sums, verticle one will come first because if you notice is more towards left and we said: read row by row from left to right. No need to consider empty cells
    * You may ask if clues are all defined in a linear string without specifying which position they were in, how would the puzzle software know where to put each number? Well, believe me it would know.
    * Example from a 10x10 Kakuro puzzle from Times of India/Mirror:
        `"0411241609221117171103221214121213041609140619232510080408200610151619171110"`

#### 2.3. Hitori (`pztype`: "hitori")

Hitori puzzles don't have a standard size specification but are generally 8x8 in Times of India. Each of the cell contains a number with no blanks. If the width of puzzle is w and height h, the puzzle will be encode as:

* **`pzlayout`**:
    * Format: A string of 4 + (w x h) characters , first 4 representing width and height as described earlier. It will be followed by numbers in the grid, read row by row, from left to right.
    * Example from a 8x8 Hitori puzzle from Times of India:
        `"08082213282424267391553277991319467827256713389312358264382411732456"`

* **`pzdata`**: `""` (empty string, not used for Hitori)

#### 2.4. Loop-the-Loop (`pztype`: "loop")

Loop-the-loop (also known as Slitherlink) puzzles involve drawing a single, continuous loop along the grid lines, with numbers in cells indicating how many of their surrounding edges are part of the loop. Loop puzzles don't have a standard size specification but are generally 7x7 in Times of India and Mirror. 

* **`pzlayout`**:
    * Format: A string of 4 + (w x h) characters , first 4 representing width and height as described earlier.
    * It will be followed by contents in the grid, read row by row, from left to right. Blank cells needs to be represented as space " "
    * Only the numbers 0-3 appear in the puzzle, other than space.
    * Example from a 7x7 Loop puzzle from Times of India/Mirror:
        `"0707   1   2  322 321 1  2 3 3    2322  202    333   "`

* **`pzdata`**: `""` (empty string, not used for Loop-the-loop)

#### 2.5. Hidato (find them at https://hidato.com or Economic Times)      (`pztype`: "hidato")

Hidato is a logic puzzle where the goal is to fill a grid with consecutive numbers that are connected horizontally, vertically, or diagonally. The grid typically starts with a few numbers already placed, and the solver must deduce the placement of the remaining numbers to create a continuous sequence from the lowest to the highest number present in the grid.

The creator site defines two types of Hidato puzzles
1. Beehive: Here the cells are of hexagonal shape and each cell connects upto 6 others.
2. Classic: Here the cells are of square shape and each cell connects upto 4 others.

The specifications here are solely for the Classic type of Hidato puzzles as they resemble the structure of other puzzles in this specification. There is no plan to include Beehive type.

While encoding the puzzle consider the puzzle as full square shaped. The puzzles appearing in the sources mentioned may remove few cells around the corners showing it as of different shape. However for encoding the entire sqare shape should be considered.

* **`pzlayout`**:

    * Format: A string of 4 + ( w x h ) characters, first 4 representing width and height as described earlier. Next set of characters represent each cell as 'x', '0' or '1'. 'x' when the sqare is darkened (i.e. won't contain any number) or cells is outside the puzzle, '0' when the cell is an answer cell to be filled by player. '1' if the cell has a number already given, and is not meant to be filled by player.
    * Example from a 9x9 Hidato puzzle from Economic Times:
        `"09090000000110111110010100101000111x0010000xxx0100001x0010010000010011110000000110110"`


* **`pzdata`**: 
    * Format: A string of characters. The number of characters will depend on how many numbers are already present in the puzzle. If their number is 'n' the length of this string would be '2xn'. A given numbers if is greater than 9 will be written as-is and if single digit it needs to be padded with 0 on left.
    * The clues need to be read row by row from left to right. If a cell contains two clues one for vertical sums and other for horizontal sums, verticle one will come first because if you notice is more towards left and we said: read row by row from left to right. No need to consider empty cells or dark cells.
    * Since the pzlayout defines which cells contain the numbers already, the value in pzdata will be accordingle put on those locations.
    * Example from a 9x9 Hitado puzzle from Economic:
        `"67690607080163681176741519204341274725483033363835565253"`



### 3. Generating a QR code for the puzzle.

To facilitate easy sharing and loading of NDro8 puzzles, the entire JSON string representing a puzzle can be encoded into a text-type QR code.

#### Encoding Process

Construct the complete JSON string: 
* Ensure the JSON string is valid and adheres to the NDro8 specification, including all required attributes and their correct formatting.
* Use a text-type QR code generator: Input the entire JSON string (from the opening { to the closing }) into any standard QR code generation tool or library that supports text encoding.

Example QR Code Input String for a Sudoku puzzle:

```
{"spec":"ndro8","ver":"1","pzname":"Sudoku : Mirror 26/05/2025","pzlayout":"0909111222333111222333111222333444555666444555666444555666777888999777888999777888999","pztype":"sudoku","pzdata":" 92 7 6 4       7   5  91     3 6   58 6     4 82   4 3     71  6   5       4 3 6 52 "}
```

### 4. Packing Multiple Puzzles into a Single QR Code

To allow for the sharing of multiple NDro8 puzzles via a single QR code, an array of puzzle definitions can be used. Given that the combined text size can become large, the array of JSON objects should be compressed and then Base64 encoded before QR code generation. A reduction of 50-65% in text length is easily achieved by this gzip/base64 arrangement.

#### Structure for Multiple Puzzles
Multiple puzzle definitions should be structured as a JSON array, where each element in the array is a complete NDro8 puzzle JSON object as defined in previous Section 
```
[
  {"spec":"ndro8","ver":"1","pzname":"Loop : Mirror 25/06/2025","pzlayout":"0707 223322323  1 302  1  2   3   13    31 2    2   3","pztype":"loop","pzdata":""},
  {"spec":"ndro8","ver":"1","pzname":"Kakuro : Mirror 25/06/2025","pzlayout":"10100000000000000110011000011101100111011100011001100000111011000001110111001101101101100011100110001100","pztype":"kakuro","pzdata":"15040915122115140907170810171124121716231608190310191512121608061111220406"},
  {"spec":"ndro8","ver":"1","pzname":"Sudoku : Mirror 25/06/2025","pzlayout":"0909111222333111222333111222333444555666444555666444555666777888999777888999777888999","pztype":"sudoku","pzdata":"53 9        8 7  13   15 4859    7        6    1394 85   69 4 7        1 45"},
  {"spec":"ndro8","ver":"1","pzname":"Hitori : Times of India 25/06/2025","pzlayout":"08082553725621957348229621369834291735134623266775894321627544993862","pztype":"hitori","pzdata":""},
  {"spec":"ndro8","ver":"1","pzname":"Hidato : Economic Times 25/06/2025","pzlayout":"09090000000110111110010100101000111x0010000xxx0100001x0010010000010011110000000110110","pztype":"hidato","pzdata":"67690607080163681176741519204341274725483033363835565253"}
]
```
#### Compression and Encoding
To minimize the data size for QR code generation, the JSON array string should first be compressed using GZIP and then encoded into a Base64 string.

Python Example for Compression and Encoding:
```
import gzip
import base64
import io

# The complete JSON string representing the array of puzzles
# (replace this with your actual multi-puzzle JSON string)
text = """
[
  {"spec":"ndro8","ver":"1","pzname":"Loop : Mirror 25/06/2025","pzlayout":"0707 223322323  1 302  1  2   3   13    31 2    2   3","pztype":"loop","pzdata":""},
  {"spec":"ndro8","ver":"1","pzname":"Kakuro : Mirror 25/06/2025","pzlayout":"10100000000000000110011000011101100111011100011001100000111011000001110111001101101101100011100110001100","pztype":"kakuro","pzdata":"15040915122115140907170810171124121716231608190310191512121608061111220406"},
  {"spec":"ndro8","ver":"1","pzname":"Sudoku : Mirror 25/06/2025","pzlayout":"0909111222333111222333111222333444555666444555666444555666777888999777888999777888999","pztype":"sudoku","pzdata":"53 9        8 7  13   15 4859    7        6    1394 85   69 4 7        1 45"},
  {"spec":"ndro8","ver":"1","pzname":"Hitori : Times of India 25/06/2025","pzlayout":"08082553725621957348229621369834291735134623266775894321627544993862","pztype":"hitori","pzdata":""},
  {"spec":"ndro8","ver":"1","pzname":"Hidato : Economic Times 25/06/2025","pzlayout":"09090000000110111110010100101000111x0010000xxx0100001x0010010000010011110000000110110","pztype":"hidato","pzdata":"67690607080163681176741519204341274725483033363835565253"}
]
"""

# Compress using GZIP
buf = io.BytesIO()
with gzip.GzipFile(fileobj=buf, mode='wb') as f:
    f.write(text.encode('utf-8'))

# Convert to Base64 string
base64_compressed = base64.b64encode(buf.getvalue()).decode('utf-8')

print(base64_compressed)
```

The base64_compressed string obtained from the above process is the final string that should be used as the input for a text-type QR code generator.




This completes the initial specification document. We can refine or add more details as needed.
