# CSafe

**Compound Passphrase List Safety Checker**

This command line tool checks whether a given passphrase word list (such as a diceware word list) has any words that can be combined to make another word on the list. It's written in Rust, which I am new to. This is very much **a toy project**, so I'd heavily caution against trusting it for real results. 

This is an updated version of [this project](https://github.com/sts10/compound-passphrase-list-safety-checker) if you want to check that out.

<!-- I also have written [a blog post](https://sts10.github.io/2018/05/05/compound-passphrase-list-safety-checker.html) about this tool. --> 


Initially I wanted to make sure that no two words in [the EFF's long diceware word list](https://www.eff.org/deeplinks/2016/07/new-wordlists-random-passphrases) could be combined to make another word on the list. The tool here is generalized to check any such word list.
## Disclosures

I am not a professional developer, researcher, or statistician, and frankly I'm pretty fuzzy on some of this math. This code/theory/explanation could be very wrong (but hopefully not harmful?). If you think it could be wrong or harmful, please create an issue! 

Further disclosures: see "Caveat" section below.

## Related projects by me that may interest you

[Tidy](https://github.com/sts10/tidy) is a Rust command-line tool that cleans and combines word lists. Notably, it can also optionally remove ["prefix" words](https://en.wikipedia.org/wiki/Prefix_code). It's my understanding that a word list that does not have any prefix words is "compound-safe", as I define that term below. And while removing all prefix words may remove more words than strictly necessary to guarantee compound-safety, it's a simpler concept to understand and check for, and thus likely better for actually vetting a word list for use without word separators than this project.

## What is "compound-safety"? 

I made up the term.

Basically, a passphrase word list is "compound-safe" (that is, it's safe to join words without punctuation or spaces) if it does NOT contain any pairs of words that can be combined such that they can be guessed in two distinct ways within the same word-length space. This includes instances in which two words can be combined and form another word on the list.

I heard of this potential issue in [this YouTube video](https://youtu.be/Pe_3cFuSw1E?t=8m36s). 

## Brief examples of compound safety violations

**Example #1**: If a word list included "under", "dog", and "underdog" as three separate words, it would NOT be compound-safe, since "under" and "dog" can be combined to make the word "underdog". A user not using spaces between words might get a passphrase that included the character string "underdog" as two words, but a brute-force attack would guess it as one word. Therefore this word list would NOT be compound-safe.

**Example #2**: Let's say a word list included "paper", "paperboy", "boyhood", and "hood". A user not using punctuation between words might get the following two words next to each other in a passphrase: "paperboyhood", which would be able to be brute-force guessed as both `[paperboy][hood]` and `[paper][boyhood]`. Therefore this word list would NOT be compound-safe. 

Another way to think about example 2: if, for every pair of words, you mash them together, there must be only ONE way to split them apart and make two words on the list. This is how I approached the issue when writing the code for csafe.

## Why is the compound-safety of a passphrase word list notable? 

Let's say we're using the word list described above, which has "under", "dog" and "underdog" in it. A user might randomly get "under" and "dog" in a row, for example in the six-word passphrase "crueltyfrailunderdogcyclingapostle". The user might assume they had six words worth of entropy. But really, an attacker brute forcing their way through five-word passphrases would eventually crack the passphrase. 

Likewise if we got the 6-word phrase "divingpaperboyhoodemployeepastelgravity", an attacker running through six-word combinations would have two chances of guessing "paperboyhood" rather than one.

**It's important to note** that if the passphrase has any punctuation (for example, a period, comma, hyphen, space) between words, both of these issues go away completely. If our passphrase is "cruelty under dog daylight paper boyhood": (1) an attacker who tries "underdog" as the third word does not get a match, (2) and the attacker likewise does not get a match if "paperboy" is guessed in the fifth slot and "hood is guessed as the sixth.

## Realistically, what are the odds of this occurring in a randomly generated passphrase?

I don't know! If you think you have a formula for calculating this on a per-list basis, feel free to submit an issue or pull request!

## What this tool does

This tool takes a word list (as a text file) as an input. It then searches the given list for compund-unsafe words.

Next, it attempts to find the smallest number of words that need to be removed in order to make the given word list "compound-safe". Finally, it prints out this new, shorter, compound-safe (csafe) list to a new text file. In this way it makes word lists "compound-safe" (or at least more safe -- see "Known issue" and "Caveat" sections below).

## How to use this tool to check a word list

First you'll need to [install Rust](https://www.rust-lang.org/en-US/install.html). Make sure running the command `cargo --version` returns something that starts with something like `cargo 0.26.0`. 

Next, clone down this repo. To run the script, cd into the repo's directory and run:

```
cargo run --release <wordlist.txt>
```

This will create a file named `wordlist.txt.csafe` that is the compound-safe list of your word list (obviously may be shorter). 

You can also explicitly specify a specific output file location:

```
cargo run --release <wordlist-to-check.txt> <output.txt>
```

### Why's it so slow? 

Yes, this is script is slow. Running the Agile word list (18k words) through it took my laptop about 26 hours. If you have speed improvements please submit an issue or pull request!

## Some initial findings

At one time, 1Password used [this list of 18,328 words](https://github.com/agilebits/crackme/blob/master/doc/AgileWords.txt) to generate passphrases for users. The list is not compound safe, though this is NOT a security issue for 1Password, the UI of which prevents users from creating passphrases without punctuation between words.

However, since it is a "real world" passphrase list and it's not compound-safe, it makes for a good demonstration for csafe. Given [that list](https://github.com/sts10/csafe/blob/main/word_lists/agile_words.txt), csafe was able to make [a compound-safe version](https://github.com/sts10/csafe/blob/main/word_lists/agile_words.txt.csafe) by only removing 1,508 words, leaving 16,820 words on the list. Note that the safer but more drastic approach of [removing all prefix words leaves you with just 15,190 words](https://github.com/sts10/prefix-safety-checker/blob/master/word_lists/agile_words.txt.no-prefix).

Again: 1Password's software, as far as I know, does NOT allow users to generate random passphrase without punctuation between words. Users _must_ choose to separate words with a period, hyphen, space, comma, or underscore. So these findings do NOT constitute a security issue with 1Password.

## Caveats / Known issues

This project only looks for "two-word compounding", where two words, mashed together, can be read in more than one way. But is there a possibility of a three-word compounding -- where three words become two? This tool does NOT currently check for this, so I can't actually guarantee that the lists outputted by the tool are completely compound-safe. This another reason to more simply remove all prefix words, as [the EFF word list creator apparently did](https://www.eff.org/deeplinks/2016/07/new-wordlists-random-passphrases). You can remove all prefix words from a list with another tool I wrote called [Tidy](https://github.com/sts10/tidy).

Also, currently this script runs pretty slowly! Using threads in Rust would help, but I'm sure there's a more efficient way to find unsafe words.

## Running tests and benchmarks

`cargo test` runs a few basic tests. 

`cargo bench` uses Criterion to benchmark the main unsafe word search function, located in `src/lib.rs`. If you're trying to help speed this project up (which would be much appreciated!) this will hopefully be useful to you.

## To do

- [ ] Use multiple threads to speed up the process. 
- [ ] Use structopt to make it a proper CLI
- [ ] Make the command line text output during the process cleaner and more professional-looking.

## Lingering questions

1. Given a word list that is not compound-safe, how can we calculate the probability of generating a non-safe pair in a passphrase of a given length (say, 6 words)?
2. Given this probability, does it make sense, or is it useful, to calculate a revised bits-per-word measure of the list? (For the record I think this would be harmful, but I pose it here for inspiration.)
3. If a word list has no prefix words, is it definitely compound-safe? Assuming yes.

