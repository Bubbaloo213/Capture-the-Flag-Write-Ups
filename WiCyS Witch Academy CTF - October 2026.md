# Women in Cybersecurity (WiCyS) Witch Academy CTF - October 2026

The objective of this exercise is to document the steps, tools, and problem-solving techniques I used to capture flags during the Witch Academy CTF, a WiCyS Cross-Chapter Capture the Flag competition involving students and cybersecurity clubs from across the country.
This repository contains my personal write-ups for challenges across several cybersecurity areas, including:

* Web Exploitation
* Password Cracking
* Digital Forensics
* Cryptography
  
Each write-up walks through my approach to solving the challenge, including how I interpreted the clues, investigated the problem, used relevant cybersecurity tools and techniques, and identified the flag.

**Disclaimer:** These write-ups are intended strictly for educational purposes and document challenges performed in an authorized CTF environment.


## Web Exploitation
### 1) The Lunar Observatory

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" height="280" alt="image" src="https://github.com/user-attachments/assets/ffe80ca6-20c8-4a4a-95a5-9edfa343c05f" />

1) Click https://the-lunar-observatory.onrender.com/

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/43907765-f600-4e66-8924-e39e9f965b81" />

#### Seeing what's behind a Webpage

2) “Seeing what’s behind a webpage” means gathering hidden information from the server’s response, code, and network traffic — not just what’s rendered in your browser.
3) View the page source by pressing **Ctrl+U** (Windows)
4) The page source shows a clue **/flag.txt**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/682a45c1-1a2b-401a-9d95-aaafd8e75c54" />

5) Add this as part of the original URL

   view-source:https://the-lunar-observatory.onrender.com/flag.txt

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/92632e8b-6c5e-4d22-8a64-c2a04be71f4c" />

**Flag:** WICYS{ph4ses_r3veal_4ll}

### 2) The Restless Wisp

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" height="250" alt="image" src="https://github.com/user-attachments/assets/e6b631d8-585b-41b2-8888-a4f04dec729b" />

1) Click https://wisp-chase.onrender.com/
2) Hover close to the main circle until it stops moving

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/911feca7-391f-4c6b-b820-c92952ee7f91" />

3) Immediately click on it and the flag will appear

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/2df26ea4-5100-4549-b4ea-44bb7dd6d06d" />

**Flag:** WICYS{st1llness_c4tches_4ll}

## Cryptography
### 1) First-Year Runes

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" height="270" alt="image" src="https://github.com/user-attachments/assets/d2ec9845-e283-4294-82d4-bc6da044abf6" />

1) Download the image

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="400" height="190" alt="image" src="https://github.com/user-attachments/assets/317699ae-7dc7-4720-acbb-7209f15f4e9a" />

2) Translate the image using an Elder Futhark reference chart: https://www.alphabetsymbol.com/runes-alphabet/?utm_source=chatgpt.com

**Flag:** WICYS{4nc13nt_run3s}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" height="300" alt="image" src="https://github.com/user-attachments/assets/3a558f11-59d6-4990-ad47-45612b320903" />

## Forensics
### 1) Whose Cat?

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="200" height="197" alt="image" src="https://github.com/user-attachments/assets/4b297150-511f-4245-88bd-ce1e1c4ee94b" />

1) Go to your Linux Machine and open the Command Line
2) Confirm what type of file is this by running:

  ```javascript
  file whose_cat.jpg
  ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="180" alt="image" src="https://github.com/user-attachments/assets/a0908516-52f9-49d6-a4c7-af0f63558ff7" />

3) The result confirms that it's a JPEG image, but is it?

4) Look at the end of the file in hexadecimal by running:
 
  ```javascript
  xxd whose_cat.jpg | tail -30
  ```
5) If it's actually a JPEG image, the end marker of the file would end in **ff d9**. Instead, the end of the file ends in **IEND.B**. **IEND** is the final chunk of a PNG file. That is excellent evidence that your JPEG contains PNG data.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="336" height="284" alt="image" src="https://github.com/user-attachments/assets/6d1c4d92-8afb-43f2-9006-a88fdc9bb25b" />

#### Find where the hidden PNG starts
6) Search the entire file for the PNG signature. PNG files begin with these bytes: **89 50 4e 47 0d 0a 1a 0a**

7) Run:
 
  ```javascript
  python3 -c "data=open('whose_cat.jpg','rb').read(); print(data.find(b'\x89PNG\r\n\x1a\n'))"
  ```
8) For this image, the PNG starts at byte: **18914 → 89 50 4E 47...**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="80" alt="image" src="https://github.com/user-attachments/assets/17e6772a-4c0d-4f49-972a-4cca085f2ef8" />

9) This indicates that the JPEG ends immediately before it at bytes: **18912 → FF D9**
10) In summary, the file is broken into JPEG + (hidden) PNG
      [ normal JPEG ]
      FF D9
      89 50 4E 47 0D 0A 1A 0A
      [ hidden PNG ]
11) Extract everything beginning at that offset (JPEG) so only the PNG extract is left. Run:

  ```javascript
  dd if=whose_cat.jpg of=hidden.png bs=1 skip=18914
  ```

  **Breaking that down:**
* ***if=whose_cat.jpg*** means input file.
* ***of=hidden.png*** means save the extracted data as hidden.png.
* ***bs=1*** means process one byte at a time.
* ***skip=58241*** means ignore the first 58,241 bytes and start copying from there.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/cfb350ab-0039-4620-b93b-63565799c3b3" />


12) Confirm that the extracted file is only the PNG portion. Run:

  ```javascript
  file hidden.png
  ```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="445" height="28" alt="image" src="https://github.com/user-attachments/assets/0f03f405-c3c1-4fb2-87ee-6f223feb2ba0" />
    
13) Result is **PNG image data**. This indicates that we successfully carved the second image out of the JPEG.
14) Open the PNG file by running:

  ```javascript
  xdg-open hidden.png
  ```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="500" height="200" alt="image" src="https://github.com/user-attachments/assets/279012ee-5bad-4577-bc82-4009a3888fba" />

**Flag:** WICYS{FAMILIAR13}

**Conclusion:** The important lesson from this CTF is that a file can contain more data than its visible file type suggests. The .jpg extension only tells you what the file is supposed to be; forensic analysis means examining the actual bytes inside it.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img width="316" height="329" alt="image" src="https://github.com/user-attachments/assets/b091c540-34b3-453b-b946-efe3c763c52f" />

