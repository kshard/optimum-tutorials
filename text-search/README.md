# Tutorial: "Text Search"

This tutorial guides you through the process of creating instances of an approximation nearest neighbor search of natural language content.

The tutorial is based on [Crime and Punishment](https://en.wikipedia.org/wiki/Crime_and_Punishment). It is a novel by the Russian author Fyodor Dostoevsky that is a psychological exploration of guilt, morality, and redemption.


## Requirements

1. Install the command-line utility from source code. It requires [Golang](https://go.dev) development environment:

```bash
go install github.com/kshard/optimum/cmd/optimum@latest
```

2. Define environment variable with values obtained from your provider. 

```bash
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=XXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXX
export HOST=https://example.com
export ROLE=arn:aws:iam::000000000000:role/example-access-role
```

3. Clone this repository, examples and tutorial section.

```bash
git clone github.com/kshard/optimum-tutorials
cd text-search
```

## Text retrieval

1. Let's create new instance of `text` data structure. We call it `scap` - Sentences of Crime and Punishment. The command displays the progress of the data structure creation. Please wait until the instance is successfully provisioned.
  
```bash
optimum text create -u $HOST -r $ROLE \
  -n scap \
  -j config.json

# Output:
# 
# scap (vsn Njqa2b7o86O72V.0) | CREATING  ...
# scap (vsn Njqa2b7o86O72V.0) | STARTING  ...
# scap (vsn Njqa2b7o86O72V.0) | RUNNING   ... 
# scap (vsn Njqa2b7o86O72V.0) | SUCCEEDED ... 
```

2. `sentences/crime-and-punishment.txt` dataset is a sequence of natural language sentences from the book. Upload this dataset.

```bash
optimum text upload -u $HOST -r $ROLE \
  -n scap \
  sentences/crime-and-punishment.txt
```

3. Commit uploaded dataset, making it available on the read path. The command displays the progress of the commit operation. Please wait until the dataset is successfully committed. Note: the commit takes about 10 min due to intentional rate limiting for text analysis.

```bash
optimum text commit -u $HOST -r $ROLE \
  -n scap

# Output
# 
# scap (vsn Njqa6b7o866fsZ.L) | COMMITTING ...
# scap (vsn Njqa6b7o866fsZ.L) | STARTING  ...
# scap (vsn Njqa6b7o866fsZ.L) | RUNNING   ... 
# scap (vsn Njqa6b7o866fsZ.L) | SUCCEEDED ... 
```

4. Check that `scap` instance is active and the update of latest version has been successful.

```bash
optimum text list -u $HOST -r $ROLE

# Output
#
# NAME  VERSION          UPDATED AT          | STATUS  PENDING |
# scap  Njqa6b7o866fsZ.L 2024-08-19 06:52:38 | ACTIVE          |
```

5. `queries.txt` dataset consists of 100 questions about "Crime and Punishment":
- Lines 01-15: Plot-Related Questions
- Lines 16-30: Character-Focused Questions
- Lines 31-50: Themes
- Lines 51-70: Philosophical Questions
- Lines 71-90: Symbolism and Imagery
- Lines 91-99. Historical and Social Context
 
```bash
optimum text query -u $HOST -r $ROLE \
  -n scap \
  queries.txt
```

The output for each query is 10 most sentences (sha1 hashes), each is ranked with the cosine distance. You can interpret is as 
* `[0.0, 0.2]` Highly similar items. Use this range when you need very close matches (e.g., finding duplicate documents).
* `(0.2, 0.5]` Moderately similar items. Useful when you want to find items that are related but not identical.
* `(0.5, 0.8]` Weakly similar items. This range could be used for exploratory results where you want to include some diversity.
* `(0.8, 1.0]` Dissimilar items. Typically, these items are unrelated, and you might filter them out unless dissimilarity is desirable (e.g., in anomaly detection).

A few examples below, shows typical output:

```
Query 1 (took 67.288517ms) | cask:vYntsiSv3o4Rga6zg7wbue4ZXR/text/scap (vsn Njzn.QG1iUgv.R.2, size 12391)
  > How does Raskolnikov’s initial crime drive the plot of the novel?
  0.103614 : the novel centers around the tormented mind of rodion raskolnikov, a man driven by desperation and ideology. the weight of raskolnikov's crime becomes an unbearable burden that consumes his soul. guilt, however, proves to be a relentless force that erodes raskolnikov’s belief in his own superiority. raskolnikov’s inner conflict represents the struggle between the desire for power and the need for human connection. raskolnikov’s interactions with others are marked by a sense of disconnection and misunderstanding. raskolnikov’s oscillation between confession and concealment reflects his deep internal conflict. raskolnikov’s crime is motivated by a mixture of desperation, pride, and a misguided sense of justice. raskolnikov’s crime is both a physical act of violence and a symbolic rejection of moral law. raskolnikov’s journey from crime to punishment is a reflection of the universal human experience of sin and redemption. raskolnikov’s eventual acceptance of his guilt is a painful but necessary step towards his redemption. 
  0.151891 : raskolnikov’s murder of the pawnbroker is both a calculated act and a fevered impulse. the novel’s pacing mirrors raskolnikov’s descent into madness, with moments of intense introspection and sudden bursts of action. 
  ...
```

```
Query 97 (took 79.000902ms) | cask:vYntsiSv3o4Rga6zg7wbue4ZXR/text/scap (vsn Njzn.QG1iUgv.R.2, size 12391)
  > In what ways does Dostoevsky portray the urban environment as contributing to moral decay?
  0.216930 : the city of st. petersburg mirrors raskolnikov’s mental state, with its oppressive atmosphere and squalor. the novel’s portrayal of st. petersburg as a city of decay and corruption reflects the moral decay of its inhabitants. 
  0.269576 : the novel explores the consequences of living according to abstract ideals without regard for human emotions and relationships. 
  0.292030 : the novel critiques the dehumanizing effects of poverty and capitalism. the novel’s depiction of urban life is bleak and oppressive, reflecting the dehumanizing effects of poverty and isolation. 
  ...
```
