# Tutorial: "Text Search and Retrieval"

This tutorial guides you through the process of creating instances of Hierarchical Navigable Small World (HNSW) for efficiently retrieving similar text.

The tutorial is based on [Crime and Punishment](https://en.wikipedia.org/wiki/Crime_and_Punishment). It is a novel by the Russian author Fyodor Dostoevsky that is a psychological exploration of guilt, morality, and redemption.

There are two variants available: the first is based on a sequence of sentences extracted from the book, while the second focuses on paragraphs. All the necessary datasets for completing both tutorials are provided.

## Requirements

1. Install the command-line utility from source code. It requires [Golang](https://go.dev) development environment:

```bash
go install github.com/kshard/optimum/cmd/optimum@latest
```

2. Define environment variable with values obtained from your provider. 

```bash
export HOST=https://example.com
export ROLE=arn:aws:iam::000000000000:role/example-access-role
```

3. Clone this repository, examples and tutorial section.

```bash
git clone github.com/kshard/optimum-tutorials
cd text-retrieval
```


## Sentence-based retrieval

The sentence-based example indexes each individual sentence from the book. 

1. Let's create new instance of `hnsw` data structure. We call it `scap` - Sentences of Crime and Punishment. The command displays the progress of the data structure creation. Please wait until the instance is successfully provisioned.
  
```bash
optimum hnsw create -u $HOST -r $ROLE \
  -n scap \
  -j config.json

# Output:
# 
# scap (vsn Njqa2b7o86O72V.0) | CREATING  ...
# scap (vsn Njqa2b7o86O72V.0) | STARTING  ...
# scap (vsn Njqa2b7o86O72V.0) | RUNNING   ... 
# scap (vsn Njqa2b7o86O72V.0) | SUCCEEDED ... 
```

2. `sentences/embeddings.txt` dataset contains embedding vectors pre-calculated using `amazon.titan-embed-text-v2` model and sha1 hashes calculated from each sentence. Upload this dataset.

```bash
optimum hnsw upload -u $HOST -r $ROLE \
  -n scap \
  sentences/embeddings.txt
```

3. Commit uploaded dataset, making it available on the read path. The command displays the progress of the commit operation. Please wait until the dataset is successfully committed.

```bash
optimum hnsw commit -u $HOST -r $ROLE \
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
optimum hnsw list -u $HOST -r $ROLE

# Output
#
# NAME  VERSION          UPDATED AT          | STATUS  PENDING | PARAMS
# scap  Njqa6b7o866fsZ.L 2024-08-19 06:52:38 | ACTIVE          | {"ef...":400, ...}
```

5. `queries.txt` dataset consists of 100 statements that characterizes "Crime and Punishment". Each of these sentence is transformed into embedding vector using `amazon.titan-embed-text-v2` model. The vector is used as a query to `hnsw` dataset.

```bash
optimum hnsw query -u $HOST -r $ROLE \
  -n scap \
  -t sentences/crime-and-punishment.txt \
  queries.txt
```

The output for each query is 10 most sentences (sha1 hashes), each is ranked with the cosine distance. You can interpret is as 
* `[0.0, 0.2]` Highly similar items. Use this range when you need very close matches (e.g., finding duplicate documents).
* `(0.2, 0.5]` Moderately similar items. Useful when you want to find items that are related but not identical.
* `(0.5, 0.8]` Weakly similar items. This range could be used for exploratory results where you want to include some diversity.
* `(0.8, 1.0]` Dissimilar items. Typically, these items are unrelated, and you might filter them out unless dissimilarity is desirable (e.g., in anomaly detection).

A few examples below, shows typical output:

```
Query (took 764.562µs) > the weight of raskolnikov's crime becomes an unbearable
burden that consumes his soul.
  0.153570 : raskolnikov felt as though something had fallen on him and was stifling him.
  0.156092 : raskolnikov’s face grew more and more gloomy.
  0.172078 : raskolnikov cried, with a feeling of anguish.
  ...
```

```
Query (took 2.982835ms) > the tension between nihilism and faith runs through
every page of the novel.
  0.304414 : “she saith unto him,” (and drawing a painful breath, sonia read
  distinctly and forcibly as though she were making a public confession of faith.)
  
  0.304971 : there are a great many nihilists about nowadays, you know, and
  indeed it is not to be wondered at.
  
  0.305200 : he was rushing away, but the passage was full of people, the doors
  of the flats stood open and on the landing, on the stairs and everywhere below
  there were people, rows of heads, all looking, but huddled together in silence
  and expectation.
  
  0.307415 : the candle-end was flickering out in the battered candlestick, dimly
  lighting up in the poverty-stricken room the murderer and the harlot who had
  so strangely been reading together the eternal book.
```


## Paragraph-based retrieval

1. Let's create new instance of `hnsw` data structure. We call it `pcap` - Paragraphs of Crime and Punishment. The command displays the progress of the data structure creation. Please wait until the instance is successfully provisioned.
  
```bash
optimum hnsw create -u $HOST -r $ROLE \
  -n pcap \
  -j config.json

# Output:
# 
# pcap (vsn NjqWi0F1zQkA.Z.0) | CREATING  ...
# pcap (vsn NjqWi0F1zQkA.Z.0) | STARTING  ...
# pcap (vsn NjqWi0F1zQkA.Z.0) | RUNNING   ... 
# pcap (vsn NjqWi0F1zQkA.Z.0) | SUCCEEDED ... 
```

2. `paragraphs/embeddings.txt` dataset contains embedding vectors pre-calculated using `amazon.titan-embed-text-v2` model and sha1 hashes calculated from each sentence. Upload this dataset.

```bash
optimum hnsw upload -u $HOST -r $ROLE \
  -n pcap \
  paragraphs/embeddings.txt
```

3. Commit uploaded dataset, making it available on the read path. The command displays the progress of the commit operation. Please wait until the dataset is successfully committed.

```bash
optimum hnsw commit -u $HOST -r $ROLE \
  -n pcap

# Output
# 
# pcap (vsn NjqWplr77y8ftV.3) | COMMITTING ...
# pcap (vsn NjqWplr77y8ftV.3) | STARTING  ...
# pcap (vsn NjqWplr77y8ftV.3) | RUNNING   ... 
# pcap (vsn NjqWplr77y8ftV.3) | SUCCEEDED ... 
```

4. Check that `pcap` instance is active and the update of latest version has been successful.

```bash
optimum hnsw list -u $HOST -r $ROLE

# Output
#
# NAME  VERSION          UPDATED AT          | STATUS  PENDING | PARAMS
# pcap  NjqWplr77y8ftV.3 2024-08-19 06:52:38 | ACTIVE          | {"ef...":400, ...}
```

5. `queries.txt` dataset consists of 100 statements that characterizes "Crime and Punishment". Each of these sentence is transformed into embedding vector using `amazon.titan-embed-text-v2` model. The vector is used as a query to `hnsw` dataset.

```bash
optimum hnsw query -u $HOST -r $ROLE \
  -n pcap \
  -t paragraphs/crime-and-punishment.txt \
  queries.txt
```

The output for each query is 10 most sentences (sha1 hashes), each is ranked with the cosine distance. You can interpret is as 
* `[0.0, 0.2]` Highly similar items. Use this range when you need very close matches (e.g., finding duplicate documents).
* `(0.2, 0.5]` Moderately similar items. Useful when you want to find items that are related but not identical.
* `(0.5, 0.8]` Weakly similar items. This range could be used for exploratory results where you want to include some diversity.
* `(0.8, 1.0]` Dissimilar items. Typically, these items are unrelated, and you might filter them out unless dissimilarity is desirable (e.g., in anomaly detection).

A few examples below, shows typical output:

```
Query (took 696.558µs) > the weight of raskolnikov's crime becomes an unbearable
burden that consumes his soul.
  0.153570 : raskolnikov felt as though something had fallen on him and was
  stifling him.

  0.182888 : raskolnikov looked gloomily at him.
  
  0.184074 : raskolnikov was so exhausted by what he had passed through that
  month that he could only decide such questions in one way; “then i shall kill
  him,” he thought in cold despair.
  
  0.184219 : this was really unbearable. raskolnikov could not help glancing at
  him with a flash of vindictive anger in his black eyes, but immediately
  recollected himself.
```

```
Query (took 1.813276ms) > the tension between nihilism and faith runs through
every page of the novel.
  0.279981 : “oh, i have. there are a great many nihilists about nowadays, you
  know, and indeed it is not to be wondered at. what sort of days are they?
  i ask you. but we thought... you are not a nihilist of course? answer me
  openly, openly!”

  0.284410 : (and drawing a painful breath, sonia read distinctly and forcibly
  as though she were making a public confession of faith.)

  0.310741 : the last phrase sounded strange in his ears. and here was something
  new again: the mysterious meetings with lizaveta and both of them--religious
  maniacs.
  
  0.314341 : under his pillow lay the new testament. he took it up mechanically.
  the book belonged to sonia; it was the one from which she had read the raising
  of lazarus to him. at first he was afraid that she would worry him about
  religion, would talk about the gospel and pester him with books. but to his
  great surprise she had not once approached the subject and had not even
  offered him the testament. he had asked her for it himself not long before his
  illness and she brought him the book without a word. till now he had not
  opened it.
```
