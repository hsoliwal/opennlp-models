<!-- SPDX-License-Identifier: Apache-2.0 -->

# OpenNLP Models fork: practical workflows

This is a navigation and command companion for `hsoliwal/opennlp-models`, a fork of
[apache/opennlp-models](https://github.com/apache/opennlp-models).
Start with the README's [model overview](../README.md#overview) and
[build-versus-training explanation](../README.md#building-model-artifacts-versus-training).
A local snapshot or newly trained model is not an official Apache release.

## Find the right layer

| Goal | Source or documentation |
| --- | --- |
| Consume an existing published model | [Dependency examples](../README.md#referencing-published-models) |
| Package a tokenizer for one language | [Tokenizer family](../opennlp-models-tokenizer) and [English leaf POM](../opennlp-models-tokenizer/opennlp-models-tokenizer-en/pom.xml) |
| Find the other task families | [Sentence detection](../opennlp-models-sentdetect), [POS tagging](../opennlp-models-pos), [lemmatization](../opennlp-models-lemmatizer), [language detection](../opennlp-models-langdetect) |
| Understand generated-JAR checks | [Validator](../opennlp-models-test/src/main/java/org/apache/opennlp/ModelValidator.java) and [expected inventory](../opennlp-models-test/src/main/resources/expected-models.txt) |
| Prepare a training experiment | [UD training resources](../opennlp-models-training/opennlp-models-training-ud/src/main/resources) and [training instructions](../README.md#training-models) |

Language-specific artifact names follow their family directory. The parent
`opennlp-models` POM aggregates these projects; it is not the Java NLP runtime.
Published dependency consumers normally do not need to clone this repository.

## Focused packaging example

With the Java and Maven versions required by the [root POM](../pom.xml), start in a
new checkout. This repository has no Maven wrapper:

```sh
git clone https://github.com/hsoliwal/opennlp-models.git
cd opennlp-models
java -version
mvn -version

# Package one model and its required reactor projects
mvn -pl opennlp-models-tokenizer/opennlp-models-tokenizer-en -am package

# Inspect the produced JAR's resource names
jar tf opennlp-models-tokenizer/opennlp-models-tokenizer-en/target/opennlp-models-tokenizer-en-1.3.1-SNAPSHOT.jar
```

The filename uses this checkout's `1.3.1-SNAPSHOT` version; adjust it if the root
POM changes. Look for the model resource declared by the selected leaf POM and its
model metadata. Listing the JAR is an inspection step, not an accuracy evaluation.
The focused command does not run the full model inventory check; use the README's
full verification command and read its documented validator limitation.

## Keep three version concepts separate

- **Artifact version:** the Maven version used to resolve or install a model JAR.
- **Model family and metadata:** the downloaded binary's family, filename, language,
  hash, and model version, declared in its leaf POM.
- **Training toolkit version:** the OpenNLP distribution selected by the training script.

These values need not be identical. Use the published artifact version you intend
to consume, and inspect its metadata rather than substituting the toolkit version
into every dependency declaration.

## Before a training run

The [training script](../opennlp-models-training/opennlp-models-training-ud/src/main/resources/ud-train.sh)
assigns its configuration variables internally. Exporting a different `OPENNLP_HOME`
will not override the assignment in the script. Review the script and adjacent
`ud-train.conf` together, including tool/treebank paths, output directory, chosen
languages, and `TRAIN_*` switches. Follow the README's preparation instructions first.

```sh
cd opennlp-models-training/opennlp-models-training-ud/src/main/resources
# Syntax check only; does not download data or train a model
bash -n ud-train.sh
```

Use Bash because the script contains arrays. Relative paths resolve from the current
working directory. Keep `CREATE_RELEASE=false` for an ordinary experiment and inspect
training/evaluation logs before reusing the result. A successful syntax check does
not establish that the required corpus files exist or that training will succeed.

## Troubleshooting by symptom

- **Download or SHA-256 failure:** inspect the leaf POM's model URL and `model.sha256`
  alongside network/proxy access. Do not bypass the digest check to accept an unexpected binary.
- **Expected/available-model mismatch:** compare the expected inventory with the
  intended artifacts and use a clean full reactor build. Old JARs from earlier builds
  can affect the validator's recursive inventory. Its current early-return limitation
  is explained in the README.
- **JAR command cannot find a file:** check that `package` succeeded, then use the
  selected module's `target` directory and the actual root-POM version.
- **Training cannot find OpenNLP or a treebank:** check the working directory and
  the script's assigned paths and language/treebank combinations before rerunning.
- **Packaging succeeds but results are poor:** inspect the correct task/language,
  training configuration, evaluation logs, and behavior on representative application
  data. Binary integrity and packaging do not measure linguistic quality.

Follow the [contribution guide](../.github/CONTRIBUTING.md) and retain applicable
model/corpus attribution described by the README. Commands and paths here were
checked against source; no build, model download, training, or evaluation was run
for this documentation.
