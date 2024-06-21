# HUS-Tokens

### 13.2 StreamTokenizer class

The last example illustrates the `StreamTokenizer` utility class. 

It reads from a text file and splits the input into ‘tokens’ – single pieces of information (treating spaces as separators, but this can be changed). 

After reading a token with the `nextToken` method, the field `ttype` contains information about the type of this token in the form of a predefined integer constant: `TT_NUMBER` if the token can be interpreted as a number, `TT_WORD` if it’s a string, `TT_EOL` if it’s the end-of-line character, `TT_EOF` if the end of file has been reached. 

When the token is a number or a string, one can get their values from the fields `nval` (`double`) or `sval` (`String`), respectively.

For example, for a data file like this

```
Tokyo 38  Delhi
25.7      Shanghai 23.7 SaoPaulo 21 Mumbai 21
```

the following program

## Listing 80 HUS-Tokens/Tokenizer.java

```java
// HUS-Tokens/Tokenizer.java
 
import java.io.BufferedReader;
import java.io.IOException;
import java.io.FileNotFoundException;
import java.io.StreamTokenizer;
import java.nio.file.Files;
import java.nio.file.Paths;
import static java.io.StreamTokenizer.TT_EOF;
import static java.io.StreamTokenizer.TT_NUMBER;
import static java.io.StreamTokenizer.TT_WORD;

public class Tokenizer {
    public static void main(String[] args) {
        double result = 0;
        StringBuilder sb = null;
        try ( // UTF8 assumed
            BufferedReader br =
                Files.newBufferedReader(
                    Paths.get("Tokenizer.dat")))
        {
            StreamTokenizer sTok = new StreamTokenizer(br);
            sTok.eolIsSignificant(false);
            sTok.slashSlashComments(true);
            sTok.slashStarComments(true);
            sb = new StringBuilder();
            while (sTok.nextToken() != TT_EOF) {
                switch (sTok.ttype) {
                case TT_NUMBER:
                    result += sTok.nval;
                    break;
                case TT_WORD:
                    sb.append(" " + sTok.sval);
                    break;
                }
            }
        } catch (FileNotFoundException e) {
            System.err.println("Input file not found");
            return;
        } catch(IOException e) {
            System.err.println("IO Error");
            return;
        }
        System.out.println("Total population: " + result);
        System.out.println("Cities: " +
                    sb.toString().substring(1));
    }
}
```

will print

Total population: 129.4
Cities: Tokyo Delhi Shanghai SaoPaulo Mumbai

## 13.3 IO cheat sheet

Let us summarize, as a reference, basic forms of reading from or writing to IO streams.

### 13.3.1 Reading/writing binary files byte-by-byte

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.IOException;
import java.io.OutputStream;

// ...

try (
    InputStream fis = new FileInputStream("fi.bin");
    OutputStream fos = new FileOutputStream("fo.bin")
){
    int n;
    while ( (n = fis.read()) != -1) {
        char c = (char)n;

    // ...
    System.out.print(c);
    fos.write(n);
  }
} catch(IOException e) { /* ... */ }
```

To make reading/writing more efficient, one may also use buffered streams:

```java
import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
// ...
    InputStream fis =
        new BufferedInputStream(
            new FileInputStream("fi.bin"));
    OutputStream fos =
        new BufferedOutputStream(
            new FileOutputStream("fo.bin"))
```

### 13.3.2 Reading a binary file into an array of bytes

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
// ...
    byte[] ba = null;
    try {
        ba = Files.readAllBytes(Paths.get("fi.bin"));
    } catch(IOException e) { /* ... */ }
    for (byte b : ba) System.out.print((char)b);
```

### 13.3.3 Reading/writing text files line-by-line

```
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import static java.nio.charset.StandardCharsets.UTF_8;
// ...
try (
    BufferedReader br =
        Files.newBufferedReader(
            Paths.get("fi.txt"), UTF_8);
    BufferedWriter bw =
        Files.newBufferedWriter(
            Paths.get("fo.txt"), UTF_8)
    ){
        String line;
        while ( (line = br.readLine()) != null) {
            System.out.println(line);
            bw.write(line);
            bw.newLine();
    }
} catch(IOException e) { /* ... */ }
```

### 13.3.4 Reading/writing text files character-by-character

One can use `BufferedReader` as in the example above but, instead of `readLine`, call `read`

```java
    // Reads one character and returns it as an int.
    // Returns -1 when the end of file has been reached
int n = br.read();
char c = (char)n;
```

To read text from the keyboard, you can use

```java
import java.io.InputStreamReader;
import java.io.Reader;
import java.io.IOException;
import static java.nio.charset.StandardCharsets.UTF_8;

  try (
      Reader rd =
          new InputStreamReader(System.in, UTF_8)
  ) {
      while (true) {
          int i = rd.read();
          // ... a condition to stop the loop
          c = (char)i;
          System.out.print(c);
      }
  } catch(IOException e) {
      e.printStackTrace();
  }
```
