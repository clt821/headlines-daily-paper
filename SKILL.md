---
name: headlines-daily-paper
description: Builds a personal daily newspaper covering the topics, city, and tone the user chose during setup, saves each edition to a folder on their own computer, and asks before treating an edition as final. Use when the user wants a personal news digest, a daily briefing, or their own newspaper, and whenever they ask to run, set up, adjust, or change anything about their Headlines edition.
---

# Headlines: Your Own Daily Personal Newspaper

A Claude scheduled task that builds you a personal "newspaper" covering only the
topics you actually care about: your interests, your city, your tone. It runs
automatically on the days and time you choose, saves every edition to a folder on
your own computer, and asks you to confirm before treating an edition as final.

This file is the instructions Claude actually follows every time the task runs.
Read README.md first if you just want to install and use it. Come back here only
if you want to understand or adjust how it works.

You only need to go through the setup below once. After that, it runs on its own
according to the days and time you chose.

## 0. First run: the setup interview

The very first time this task runs for a new user, or any time
Config/preferences.md does not exist yet, run this short interview before
building anything. Ask it as a normal conversation, not a form. One or two
questions at a time is fine. Save the answers to Config/preferences.md in plain
language (see the template in config/preferences.example.md) and use that file
on every later run.

The required questions below must be answered before the first edition can be
built, since the report genuinely cannot be assembled without them. The
optional questions can be skipped or answered later, and Claude should use the
fallback defaults in section 1 for anything left unanswered.

Required questions, ask these in order:

1. What topics or beats do they want covered? A short list, about three to six
   items. Examples: climate policy, indie games, a specific sports team, home
   renovation, a particular industry.
2. What city should the weather and almanac details reflect? This is required
   even if the person has no interest in local news coverage itself, since the
   masthead and almanac strip need a location.
3. What time should this run, and on which days? Offer three simple choices:
   every day, weekdays only, or a custom set of days they specify.
4. Where should editions be saved? Ask for a folder on their own computer.
   Saving locally is what lets Claude remember the paper's look from one
   edition to the next and keep track of which stories have already been
   reported so nothing repeats. Email delivery is not available yet. That may
   come in a later version, but for now everything is saved to the local
   folder they choose.

Optional questions, ask these after the required ones, in any order:

5. What should the paper be called? The default name is "Headlines." Ask if
   they would like to keep that name or change it. If they want to change it,
   offer a few alternatives such as "The Daily Brief," "Morning Edition," "The
   Rundown," "Your Daily Read," or "The Briefing," and also let them supply
   their own title if none of those fit.
6. Who is this for, and what is their world? A name or how they would like to
   be addressed, and a one line description of their work, field, or
   interests. This sets a general sense of what kinds of topics to prioritize
   when choosing news.
7. Are there any specific sources they trust or want prioritized? If skipped,
   Claude uses its normal judgment about reputable sources for each topic.
8. Are there any sources they would rather Claude not use at all? Keep a plain
   list of excluded sources and never pull from them.
9. Would they like a dedicated local news section for the city given above,
   real local stories and things to do, or would they rather leave local news
   out of the report entirely? This is separate from the city itself, which
   is required regardless.
10. How long should reading it take, and what tone? The length is entirely up
    to how much they choose to include. Some editions will be quick to read,
    others will run longer if there is a lot going on in their chosen topics,
    and that is expected rather than a problem to fix. Ask whether they want a
    short "why it matters" line on major stories or just the facts.
11. Do they want either of the optional daily puzzles? Offer three choices: a
    daily sudoku only, a daily word search only, or have it rotate between
    the two. If they choose sudoku, word search, or both, also ask what
    difficulty level they want, easy, medium, or hard, for each puzzle type
    they chose. Also ask what accent color they would like for the design.
    The default is indigo.

Once the required questions are answered, before the scheduled task is set up
to run on its own, build one full trial edition right away in this same
conversation, following the normal run steps in section 2. Present it the
same way any edition is presented, and say plainly that this is a trial run
so they can check that everything looks right and make any adjustments
before the recurring schedule takes over. Invite specific tweaks, tone,
length, which topics landed well, the design, the puzzle, anything else,
and update Config/preferences.md right away for whatever they ask to change.
Only set up the actual recurring schedule once they are satisfied with this
trial edition, or once they say it is fine to proceed as is.

Preferences can be revised at any time. If the person says "actually, drop
tennis" or "make the tone more casual," update Config/preferences.md right away
and confirm the change.

## 1. Fallback defaults, used for anything not set

| Setting | Default |
|---|---|
| Paper name | Headlines |
| Topics or beats | General world and national news, business and technology, one culture or entertainment item |
| City for weather and almanac | Ask again before the first run rather than guessing one; this is the one setting that should not be silently defaulted |
| Local news section | Off, city is still used for the almanac even when this is off |
| Length and tone | As long as the chosen topics warrant, neutral, plain language, a "why it matters" line on major stories only |
| Sources | Claude's normal judgment on reputable, mainstream sources |
| Excluded sources | None |
| Save location | A folder the user names during setup; if truly skipped, ask again before the first run rather than guessing a path |
| Puzzle | Daily sudoku, medium difficulty |
| Accent color | Indigo (#4b45c4, dark #312a8c, tint #e9e7fb) |
| Schedule | Every day, weekday mornings, in the user's own time zone |

A run using all defaults must still produce a complete, properly formatted
edition, never a blank or broken page. If a section genuinely has nothing fresh
to report, say so plainly in that section rather than leaving it out silently.

## 2. What this task actually does, the mechanism, not the preference

Every run does the same job regardless of whose preferences are loaded.

1. Read Config/preferences.md for this user's topics, tone, length, city,
   excluded sources, and puzzle choice. Newer dated notes in that file win over
   older ones and over the defaults above.
2. Read Config/dedup_log.json, a plain list of entries for stories already
   reported, across all runs, going back 14 days.
3. Open the most recently built edition in the user's chosen folder and match
   its HTML and CSS structure exactly, so the visual design stays consistent
   from day to day. If none exists yet, build fresh using the design system in
   section 7.
4. Research and assemble the sections in section 5, honoring this user's
   topics, tone, length, and excluded sources, and the freshness and duplicate
   checking rules in sections 3 and 4.
5. Build the HTML edition, then render it to a matching PDF.
6. Do the required end of run writes in section 9.
7. Present the built edition as an unsaved draft and ask the person to confirm
   before it is treated as final. It has already been saved to a drafts folder
   at this point, so nothing is lost either way. Confirming simply moves the
   pair into the permanent folder using the same file naming convention.

The rest of this document explains how each of those steps actually works.

## 3. Freshness rules, essential, applies to every user

* Only include items from the last 24 to 48 hours unless the user's preferences
  say otherwise. Search with recency in mind and confirm each item's actual
  publish date. Do not assume something is fresh just because of where a link
  appeared, since a "related articles" list can surface old evergreen content.
* When in doubt, leave it out and say so plainly rather than guessing.
* Print the source name and date under each item.
* No editorial bias. Report what happened without value judgments. Keep any
  "why it matters" framing to plain significance.
* Never pull from a source the user has asked to exclude.

## 4. Checking for duplicates, applies to every user

* Before including an item, check Config/dedup_log.json. If the same story ran
  in the last 7 days and nothing new has happened, leave it out. Only bring it
  back once there is a genuine new development, and say what changed.
* Append every item actually reported today to Config/dedup_log.json, with a
  date, a short key, and a one line summary, at the end of every run, and
  remove entries older than 14 days.

## 5. Content sections, fixed order, the shape is mechanism, the content is preference

The shape below, a masthead, a short orienting strip, a handful of news
sections, and a closing note, is what every edition looks like. Which topics
fill each section comes entirely from that user's preferences.

1. Masthead: the paper's name, today's date, a one line tagline, and the user's
   city.
2. Almanac strip: a compact row showing weather high and low for the user's
   city, moon phase, and the day of year. The city here comes from the setup
   interview and is always shown, even for a user who declined a full local
   news section. Keep it small.
3. Morning thought: one short, verified, attributed quote in a tinted banner.
   Rotate daily, and try not to repeat a quote used in the last 30 days.
4. The Day Ahead: two or three sentences orienting the reader to what matters
   most today, across everything below.
5. Top Stories: the two or three biggest stories in the user's general news
   topics. Plain significance only, no forced personal angle.
6. Local, only if the user opted into a dedicated local news section during
   setup: real local news plus a short "things to do or notable openings"
   line. This is separate from the city used for weather above, a user can
   have one without the other.
7. Topics You Follow: one subsection per topic or beat the user named at
   setup, including any recurring interest such as a sports team, a market, or
   a hobby. Report the latest development plainly for each, and note whether
   something is happening specifically today.
8. Social and culture: what is moving in the user's stated interests on social
   platforms, when relevant, plus top general culture news. Do not pad this
   section with historical trivia, that belongs only in the fun fact section.
9. Fun fact of the day: one genuinely interesting, verified true fact, tied to
   the user's world or the date when possible, with its source. This is the
   only place for "on this day" history.
10. Daily puzzle, optional module: a sudoku, a word search, or whichever one
    the rotation calls for today, built at the difficulty level the user
    chose during setup, playable on screen and printable, with the solution
    on its own page.
11. Closing note: one line inviting feedback, such as "reply to tweak anything
    about this," and the save prompt from section 8.

## 6. Adjusting length by tone setting

* Quick to read: Top Stories capped at two items, Topics You Follow capped at
  one short paragraph per topic, no more than four sections total beyond the
  masthead block.
* Standard, the default: as described in section 5 in full.
* Thorough: allow a third Top Story and two or three sentences per topic
  subsection instead of one.

## 7. Design system, the mechanism is consistency, the specific palette is preference

* For a brand new user's very first edition, start from
  reference/design_template.html in this repository and copy its CSS
  verbatim. That file is a working example of the real design this skill is
  built around, a clean white sheet on a soft gray background, a large bold
  masthead with the second word of the name in the accent color, a thin
  three cell almanac strip, a tinted quote banner, an accent colored left
  border on the lead paragraph, small uppercase pill shaped tags above each
  section headline, hairline dividers between items, and a fully interactive
  sudoku with a canvas based confetti celebration on completion. Do not
  redesign or simplify this template, it already reflects real design work
  and should be treated as the standard, not a rough draft to improve on.
* For every edition after the first, match the previous edition's actual
  HTML and CSS exactly for continuity. Do not restyle between runs. Only
  change the design when the user explicitly asks for it.
* The reference template's tag colors are named generically, tag.general for
  Top Stories, tag.local for Local, tag.topic-a through tag.topic-d to cycle
  through the user's Topics You Follow subsections, tag.social for Social and
  Culture, tag.fun for the Fun Fact, and tag.puzzle for the Daily Puzzle. Use
  these as they are for any user who has not asked for a different palette.
* Default accent colors, taken directly from the reference template: accent
  #4b45c4, dark #312a8c, tint #e9e7fb. If the user chose a different accent
  color at setup, swap the three accent values consistently everywhere but
  keep the same layout, spacing, and every other color untouched.
* Include a print stylesheet so the HTML prints cleanly to the PDF, as the
  reference template already does.
* Sudoku cells show heavier 3x3 box borders, matching the reference
  template's table.sudoku rules exactly.
* Word search grids, if that puzzle is active, should let the reader click or
  drag to select a run of letters and should support the same Check, Reveal,
  and Clear controls as the sudoku, plus the same canvas based confetti
  celebration on completion, reusing the celebrate function from the
  reference template rather than writing a new one.
* Any icons, weather, moon, fun fact, or special day icons, must be embedded
  as inline base64 image data, not literal emoji characters, exactly as the
  reference template does. PDF rendering environments frequently lack a
  color emoji font and literal emoji can render blank or flat. The reference
  template already includes working sun, moon, calendar, and light bulb
  icons ready to copy as they are; only source a new icon from a small open
  icon set if a genuinely new one is needed, and keep it in Config so later
  runs reuse it without resourcing.

## 8. Draft first saving, mechanism, applies to every user

* Build every edition into a Drafts folder inside the user's chosen save
  location, never directly into the permanent folder. Name files with the
  paper's name and today's date using underscores, for example
  Headlines_2026_09_05.html and Headlines_2026_09_05.pdf.
* Writing to the Drafts folder is not the same as calling an edition final, and
  it needs no confirmation. Do it every run.
* Present the draft, both HTML and PDF, to the user and explicitly ask them to
  confirm before it is copied into the permanent folder. If they never
  confirm, that is fine, it simply stays in Drafts, which is a normal outcome
  and not an error.
* Only once the user confirms, in the same run or a later message, copy the
  pair into the permanent folder using the same naming convention.
* Never delete unconfirmed drafts yourself. If Drafts holds more than 14
  unconfirmed editions, say so plainly at the end of the run and let the user
  decide what to clean up.
* Convert the finished HTML to a PDF with a real HTML to PDF renderer such as
  WeasyPrint. Confirm the PDF actually rendered, a reasonable size and page
  count, and that any puzzle or icons rendered as expected, before presenting
  it.
* Verify the puzzle before it goes into the edition rather than trusting that
  it came out right. For a sudoku, two things have to hold: every clue printed
  on the grid matches the answer key the checker grades against, and the
  puzzle has exactly one solution. Confirm both by solving the grid
  programmatically, counting the solutions, and comparing each printed clue
  against the key. A grid whose clues contradict the key has no valid
  solution at all, and a grid with more than one solution will tell a reader
  who solved it correctly that they are wrong. For a word search, every word
  on the list has to actually appear in the grid, in the position the reveal
  points to. If a puzzle fails any of these checks, build a new one and check
  it again. Never ship a puzzle that has not passed.

## 9. End of run writes, mechanism, do every run regardless of the save decision

Do these before presenting the draft, not after. A run that "finished" but
never did these writes is not actually finished.

1. Append today's reported items to Config/dedup_log.json, and remove entries
   older than 14 days.
2. If any optional integration the user has set up requires a companion file
   update, do that update now, every time, without waiting on confirmation.
3. Reopen whatever was just written and confirm it actually landed, the right
   date, the right links, before telling the user it is done.

Then present the draft, for example through a file sharing tool, with a two or
three line summary of what is actually in today's edition, and ask for
confirmation before treating it as final. If any section had no fresh,
trustworthy news, say so plainly rather than inventing content.

## 10. Scheduling

This is meant to run as a recurring scheduled task, at the time and on the days
the user chose during setup, every day, weekdays only, or a custom set of
days. Set the schedule using whatever native scheduled task feature is
available in the environment running this skill, pointed at this file's
instructions. Nothing else in this skill depends on a specific scheduler, it
only needs to be invoked once a day, on the chosen days, with these
instructions and access to the Config and save location folders described
above.

## 11. Optional integrations, off by default, entirely preference side

Some users build small additions around their own edition of this paper, a
live "in progress" companion view, a shared personal dashboard card, and so
on. None of these are required for the core skill to work, and none should be
built unless the user asks for one. If a user wants one, treat it as its own
small addition, read their existing file or format exactly as documented
wherever they point you, write only the parts they have asked Claude to own,
and never touch parts of a shared file that belong to some other task or tool.

## 12. Quality bar for every run

* Never invent content. If a section has nothing fresh and trustworthy, say
  so.
* Never quietly skip one of the end of run writes in section 9 to save time.
  If a run is taking too long, cut research depth first.
* Keep the whole edition readable within the time the user asked for, whether
  that ends up being a quick read or a longer one.
* Spell out unfamiliar acronyms on first use.
