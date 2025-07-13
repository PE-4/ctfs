
<img width="968" height="105" alt="image" src="https://github.com/user-attachments/assets/84ec1d6d-3efb-4711-9268-2d3fac1dbe35" />


# Part 1: Find the hard-coded credential in the application used to encrypt log files. (format jUStHEdATA, no quotes)

If you haven't been under a rock, you know this is talking about the TeleMessage hack, so I started by googling:`telemessage hardcoded credential`.

That led me to this [Reddit thread](https://www.reddit.com/r/technology/comments/1ke44yl/heres_the_source_code_for_the_unofficial_signal/), where someone had posted a link to the source code of an unofficial Signal client by TeleMessage: https://github.com/micahflee/TM-SGNL-Android

From the screenshot in the Reddit post, I saw a file path that looked like this:
`blob/libs/app/src/tm/java/org/archiver/ArchiveConstants.kt`

I opened that file in the GitHub repo and found the credential `enRR8UVVywXYbFkqU#QDPRkO` on line 46:
<img width="555" height="47" alt="image" src="https://github.com/user-attachments/assets/e44987a9-75db-4e3b-8eca-4fada6b4c155" />

# Part 2: Find the email address of the developer who added the hard-coded credential from the question one to the code base (format name@email.site)
I started by cloning the repo: `git clone https://github.com/micahflee/TM-SGNL-Android`

To find out when it was introduced, I used the `-S` with `git log` to search for the credential: `git log -S enRR8UVVywXYbFkqU#QDPRkO`

Aaaand boom:

<img width="599" height="79" alt="image" src="https://github.com/user-attachments/assets/16cdbde4-be15-4b20-8920-e0c3a253f2c5" />

The email associated with the commit: `moti@telemessage.com`

# Part 3: Find the first published version of the application that contained the hard-coded credential from question one (case sensitive, format Word_#.#.#......).
We got the version tag from the last command `git log -S enRR8UVVywXYbFkqU#QDPRkO`, which showed it was first introduced in: `Release_5.4.11.20`

# Final Answers:
- Part 1: `enRR8UVVywXYbFkqU#QDPRkO`
- Part 2: `moti@telemessage.com`
- Part 3: `Release_5.4.11.20`
