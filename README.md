# Knowledge base intertwined with personal information
last_modified: 2025-08-20 17:55:29 +0000
*for potential stalkers haha*

## Sections

1. **About**  
   Information and hyperlinks to other profiles.  
   Include Twitter, Medium, and GitHub profiles.

2. **Notes**  
   migrate simple notes that are already on HackMD and other similar profiles.  
   starter guides to avoid sharing redundant information multiple times through one-on-one interactions.

3. **Where should you go next**  
   Includes itineraries for places I've already been to.

4. **Travel plans**  
   Compiled from JEE era, scenic train routes. Might not cover them in the near future.

5. **Should you watch this**  
   I dont watch a lot of movies, but when I do I do conjure up a review right away.
   These were stored locally on device till 12/25, and was one of the primary reasons why I wanted to have a knowledge base.
   Note that some (older) reviews might not have ratings, as I did not collect star-review data right away for those items.  
   With a heatmap to visualize how often I binge watch.

---

That would be enough for now, will add more stuff if I get inspired by cooler stuff outside.

---

## Instructions

- When updating config:  
  `bundle exec jekyll clean`

- To run locally:  
  `bundle exec jekyll serve`

- Run python scripts:  
  `python addReviews.py`

## Setup repo

For mac-os, install the venv manager for ruby using 
`brew install rbenv ruby-build`

Add `eval "$(rbenv init - zsh)"` to your zshrc profile by running `nano ~/.zshrc` in your terminal.

Hit rbenv install 3.3.0 (or basically any version above 3.x as used by github pages)
Set this version for local use using `rbenv local 3.3.0` then verify by running ruby -v

`bundle install` for the very first time