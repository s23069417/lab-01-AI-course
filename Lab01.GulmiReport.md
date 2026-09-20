**Lab 01 - The price of one request**

**Student: Kabylkhanova Gulmira**

**1.Forecast and measured value**

I compared the number of tokens in the same bank customer complaint in
three languages (using the Claude Opus 5 model).

  -----------------------------------------------------------------------
                          My forecast             Measured
  ----------------------- ----------------------- -----------------------
  RU/EN                   1.9                     1.46(134 tokens vs. 92)

  KK/EN                   2.3                     2.15(198 tokens vs. 92)
  -----------------------------------------------------------------------

I recorded the forecast before launching Part 2 (measurement via API),
relying on the table from Part 1: Russian and Kazakh texts take up
approximately twice as much space as English, since Cyrillic letters are
stored in 2 bytes, while Latin letters are stored in 1 byte (for the
complaint, this amounts to 1.92 for Russian and 2.13 for Kazakh).
Forecast: RU/EN 1.9 and KK/EN 2.3. Measured on Claude Opus 5: RU/EN 1.46
(134 tokens vs. 92) and KK/EN 2.15 (198 vs. 92). I estimated the Kazakh
almost accurately, but I overestimated the Russian: the Claude tokenizer
turned out to be gentler with Russian than the bytes predict. It's
impossible to assess based on words and characters: the Kazakh complaint
has fewer words (42 vs. 54), almost the same number of characters (344
vs. 300), and twice as many tokens. Different models calculate
differently: Haiku 4.5 gives the same complaint RU/EN 2.05 and KK/EN
3.02, because its English complaint is shorter (66 tokens).

**2. Annual cost**

I took a volume of 5,000 requests per day, i.e., 1.825 million per year:
this is an assumption for the support queue of a large bank, and the
calculation is linear, so for a different volume, the amounts are simply
multiplied. The prices per million tokens are taken from prices.py. Each
model is calculated based on its own measurement: its own input tokens
and its own response length.

  -----------------------------------------------------------------------
  Model             EN                RU                KK
  ----------------- ----------------- ----------------- -----------------
  Haiku 4.5         \$1 035           \$3 849           \$4 822

  Sonnet 5          \$8 395           \$18 976          \$15 246

  Opus 5            \$40 880          \$52 049          \$46 556
  -----------------------------------------------------------------------

The Kazakh query is 2.19 times more expensive than the English one in
terms of input tokens (Opus and Sonnet). However, the final bill grows
less: by 1.14 times for Opus, by 1.82 times for Sonnet, and by 4.66
times for Haiku. These are different numbers: the bill also depends on
the length of the response, and output tokens cost 5 times more than
input tokens (for Sonnet, they account for about 92% of the cost of the
Kazakh query).

**3. Which model should be put on the queue in Kazakh?**

I would put Sonnet 5.

In the complaint, the client writes that they attached the contract and
statement, but they don't have them, and the system prompt allows
responding only based on the client's documents. This means that a good
response shouldn't invent a reason for reducing the rate; it should ask
for the documents. Sonnet and Opus do exactly that in Kazakh: they say
that there are no documents and ask for the contract and statement.
Haiku lists "typical reasons" for reducing the rate (including taxes and
fees), i.e., it comes up with things it doesn't know.

In terms of price, Sonnet costs \$15,246 per year in Kazakh, Opus costs
\$46,556, and Haiku costs \$4,822. Sonnet is three times cheaper than
Opus for the same result. Haiku is even three times cheaper, but the
savings are deceptive: a response with a made‑up reason violates the
system prompt requirement. The conclusion is based on one response per
language from each model, so it needs to be verified with a larger
number of complaints.

**4. Cost lever not used in the lab**

I would limit the length of the response in the system prompt: on
Sonnet, about 92% of the cost of a Kazakh query is due to output tokens,
which are 5 times more expensive than input tokens, so a shorter
response reduces the bill more significantly than any reduction in the
question. I havent measured this lever.

**AI-use declaration**

I used Claude (Anthropic, via the claude.ai chat) to explain the essence
of the task and its conditions, to help set up Python, VS Code and
GitHub, and to resolve errors in the terminal and in working with Git.
While I was looking for a way to get an API key, Claude prepared an
alternative version of the scripts for the Google Gemini API, but I
didnt use it in the final measurements.

I ran the lab original scripts, without making any changes to the code,
using my own API key from Anthropic: part0_tokenizers.py and
part1_offline.py in offline mode, part2_measure.py for claude-opus-5,
claude-sonnet-5 and claude-haiku-4-5 (token counting and \--call), as
well as part3_cost.py. The data on the number of tokens and costs in
this report is taken from the logs in my repository (count-log.txt,
run-\*.txt, cost-\*.txt, measurements-\*.json), not from online sources.
I personally read three responses from each model. My prediction was
made before the release of the second part, based on the byte table from
the first part. My API keys are stored only in a local .env file, which
is not in the repository and I deleted them after completing the work.
