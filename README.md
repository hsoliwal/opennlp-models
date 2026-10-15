<!--
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to You under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<!-- fork-homepage:start -->
# OpenNLP Models: choose and trace the model behind a result

**Versioned language-model resources for OpenNLP tokenization, sentence detection, POS tagging, lemmatization, and language detection.** This is the public `hsoliwal` fork of [apache/opennlp-models](https://github.com/apache/opennlp-models). Use it to find concrete artifacts, understand how downloaded binaries become JAR resources, and keep packaging, training, and linguistic evaluation separate.

## What this repository provides

- **Model artifacts:** language-specific resource JARs grouped by task, plus a language-detection model. The [artifact inventory](MODEL_INVENTORY.md) maps the inspected 145 model-bearing artifacts across 153 POM projects.
- **Packaging definitions:** URLs, binary filenames, metadata, and SHA-256 values used to download and package existing models.
- **Training resources:** a UD training script and configuration that need an external OpenNLP CLI and separately obtained treebanks.
- **A packaging validator:** Java tooling that checks expected resources, with an important early-return limitation explained below.

This fork contributes navigation and source-backed explanations. It does not claim newly trained models, improved accuracy, or an implemented M3 adapter. The inspected artifact version is `1.3.1-SNAPSHOT`; do not assume that this development version is available from Maven Central. Published dependency versions must be selected separately. Original upstream material is retained below.

## Key concepts

| Term | Meaning | Practical consequence |
| --- | --- | --- |
| Task / language | What a model predicts and the language it was trained for | Select both; a POS model is not a tokenizer |
| Model binary | Learned parameters in a `.bin` resource | The OpenNLP runtime must load the correct model type |
| Resource JAR | Package containing a model and metadata | It supplies data, not a new inference engine |
| UD treebank | Universal Dependencies annotated language data | Training needs matching corpus conventions and license review |
| Checksum | Identity/integrity check for downloaded bytes | Matching bytes does not establish prediction quality |
| Evaluation | Measurements on held-out examples | Report dataset, runtime, model identity, metric, and limits |

## Quick start: package one English tokenizer

Use Java 17+ and Maven 3.6.3+ as specified in the [root POM](pom.xml). Building downloads the declared model binary and Maven dependencies.

```sh
git clone https://github.com/hsoliwal/opennlp-models.git
cd opennlp-models
git rev-parse HEAD
java -version
mvn -version
mvn -pl opennlp-models-tokenizer/opennlp-models-tokenizer-en -am package
```

This focuses on one artifact and its reactor prerequisites. It does not train a new model or run the full repository validation. The [English tokenizer POM](opennlp-models-tokenizer/opennlp-models-tokenizer-en/pom.xml) identifies the model resource as `opennlp-en-ud-ewt-tokens-1.3-2.5.4.bin`.

## Worked example: check the package you produced

After the selected build succeeds, inspect the current snapshot's JAR from the checkout root:

```sh
jar tf opennlp-models-tokenizer/opennlp-models-tokenizer-en/target/opennlp-models-tokenizer-en-1.3.1-SNAPSHOT.jar
```

Look for the declared `.bin` resource and `model.properties`. If the project version changes, use the JAR actually produced under that module's `target` directory. This inspection answers “was the expected resource packaged?” To answer “does it tokenize my data well?”, load it using the matching runtime and evaluate representative labeled examples. Those are separate steps, not results established by this documentation update.

## Compact cheat sheet

| Goal | Command / source |
| --- | --- |
| Package the complete reactor | `mvn clean package` |
| Reach configured packaging verification | `mvn clean verify` |
| Select a concrete task/language | [Model artifact inventory](MODEL_INVENTORY.md) |
| Review the expected resources | [expected-models.txt](opennlp-models-test/src/main/resources/expected-models.txt) |
| Syntax-check training script without training | `bash -n opennlp-models-training/opennlp-models-training-ud/src/main/resources/ud-train.sh` |
| Review training settings | [ud-train.conf](opennlp-models-training/opennlp-models-training-ud/src/main/resources/ud-train.conf) |
| Check a local edit | `git diff --check` |

**Validation caveat:** [ModelValidator](opennlp-models-test/src/main/java/org/apache/opennlp/ModelValidator.java) compares artifact counts but returns from `main` after its first successful expected-resource match. A successful invocation must not be represented as exhaustive validation of every binary. Packaging validation also does not load models or measure linguistic accuracy.

If a download fails or its checksum differs, check the exact leaf POM and resource URL before changing the expected hash. If training cannot find its inputs, inspect the script's `OPENNLP_HOME` and `UD_HOME`: training resources live under `opennlp-models-training/opennlp-models-training-ud/src/main/resources/`, and require the external tools/data named there. No download, build, training, inference, or evaluation was executed for this homepage update.

**Go deeper:** [packaging, training, and API guide](FORK_GUIDE.md) · [complete model inventory](MODEL_INVENTORY.md) · [training script](opennlp-models-training/opennlp-models-training-ud/src/main/resources/ud-train.sh) · [license](LICENSE) · [notices](NOTICE).

## Preserved upstream project documentation

<!-- fork-homepage:end -->

Welcome to Apache OpenNLP Models!
===========

[![GitHub license](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://raw.githubusercontent.com/apache/opennlp-models/main/LICENSE)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/org.apache.opennlp/opennlp-models/badge.svg)](https://maven-badges.herokuapp.com/maven-central/org.apache.opennlp/opennlp-models)
[![Build Status](https://github.com/apache/opennlp-models/workflows/Java%20CI/badge.svg)](https://github.com/apache/opennlp-models/actions)
[![Contributors](https://img.shields.io/github/contributors/apache/opennlp-models)](https://github.com/apache/opennlp-models/graphs/contributors)
[![GitHub pull requests](https://img.shields.io/github/issues-pr-raw/apache/opennlp-models.svg)](https://github.com/apache/opennlp-models/pulls)
[![Stack Overflow](https://img.shields.io/badge/stack%20overflow-opennlp-f1eefe.svg)](https://stackoverflow.com/questions/tagged/opennlp)

The Apache OpenNLP library provides binary models for processing of natural language text. 
This repository is intended for the distribution of model files as a Maven artifacts.

## Useful Links

For additional information, visit the [OpenNLP Home Page](https://opennlp.apache.org/models.html).

You can use OpenNLP with many languages. Additional demo models are provided [here](https://opennlp.sourceforge.net/models-1.5/).

The models are fully compatible with the latest [OpenNLP release](https://opennlp.apache.org/download.html). They can be used for testing or getting started.

> [!NOTE]  
> Please train your own models for all other, specialized use cases.

Documentation, including JavaDocs, code usage and command-line interface examples are available [here](https://opennlp.apache.org/docs/)

You can also follow our [mailing lists](https://opennlp.apache.org/mailing-lists.html) for news and updates.

## Overview

We provide **Tokenizer**, **Sentence Detector** and **Part-of-Speech Tagger** models for the following 36 languages:

   - Afrikaans
   - Armenian
   - Basque
   - Bulgarian
   - Catalan
   - Croatian
   - Czech
   - Danish
   - Dutch
   - English
   - Estonian
   - Finnish
   - French   
   - Georgian
   - German
   - Greek
   - Icelandic
   - Indonesian
   - Irish
   - Italian
   - Kazakh
   - Korean
   - Latvian
   - Norwegian
   - Persian
   - Polish
   - Portuguese
   - Romanian
   - Russian
   - Serbian
   - Slovak
   - Slovenian
   - Spanish
   - Swedish
   - Turkish
   - Ukrainian

These models are compatible with OpenNLP `>= 1.0.0`. Further details are available at the [OpenNLP Models](https://opennlp.apache.org/models.html) 
page and in the [CHANGELOG](https://dist.apache.org/repos/dist/release/opennlp/models/ud-models-1.3/CHANGES).

In addition, we provide a **Language Detector**, which is able to detect 103 languages in ISO 693-3 standard. 
Works well with longer texts that have at least 2 sentences or more from the same language. 

It is compatible with OpenNLP `>= 1.8.3`. Model details are available [here](https://downloads.apache.org/opennlp/models/langdetect/1.8.3/).

## Getting Started

The [Universal Dependencies](https://universaldependencies.org) (UD) community provides a framework for consistent annotation of grammar across different human languages.
The project is developing cross-linguistically consistent treebank annotation for 150+ languages.           

### Referencing published Models

You can import UD-based model artifacts directly via Maven, SBT or Gradle, for instance:

#### Maven

```
<dependency>
    <groupId>org.apache.opennlp</groupId>
    <artifactId>opennlp-models-pos-de</artifactId>
    <version>${opennlp.models.version}</version>
</dependency>
```

for all **36** supported languages, listed on the Apache OpenNLP [Model page](https://opennlp.apache.org/models.html).

The broader langdetect model can be referenced like this:   

```
<dependency>
    <groupId>org.apache.opennlp</groupId>
    <artifactId>opennlp-models-langdetect</artifactId>
    <version>${opennlp.models.version}</version>
</dependency>
```

#### SBT

```
libraryDependencies += "org.apache.opennlp" % "opennlp-models-langdetect" % "${opennlp.version}"
```

#### Gradle

```
compile group: "org.apache.opennlp", name: "opennlp-models-langdetect", version: "${opennlp.version}"
```

For more details please check our [documentation](https://opennlp.apache.org/docs/)


### Training Models

All released _sentence detection_, _tokenization_, _lemmatizer_, and _POS tagging_ models were and can be trained via the `ud-train.sh` script.
It is located in the _opennlp-models-training-ud_ directory in this repository. 

#### Preparing the environment

Before training UD-based OpenNLP models, the execution environment needs the latest [OpenNLP release](https://opennlp.apache.org/download.html) and the latest set of [UD treebanks](https://universaldependencies.org/#download).
Download the corresponding archive files and uncompress them both in the same directory in which the training script resides.
Rename both folders according to the `OPENNLP_HOME` and `UD_HOME` variables. 

> [!IMPORTANT]
> Check and adjust the version string in both variables, that is, to the versions you have actually downloaded. 

#### Selecting model types

Next, select what type of models should be trained. By default, the script defines:

```
TRAIN_TOKENIZER="true"
TRAIN_POSTAGGER="true"
TRAIN_SENTDETECT="true"
TRAIN_LEMMATIZER="true"
```

Simply switch off a certain type, by setting the corresponding variable to false.

#### Selecting languages

By default, treebanks of 36 supported languages are included in the `MODELS` variable of the script.
If only a smaller or different (sub-)set is required, this variable can simply be edited.
The format must be followed: `<Language>|<2-digit-locale-code>|<UD treebank name>`, for example: `English|en|EWT` or `Swedish|sv|Talbanken`.

> [!NOTE]
> The full list of supported languages and related treebanks is available [here](https://universaldependencies.org/#current-ud-languages).
> Yet, even listed on the UD page, training OpenNLP models might not succeed. If it succeeds, check the evaluation logs (_*.eval_) if the computed accuracy meets your expectations.
                       
#### Adjusting training parameters

Once you're done with the preparations, check the `ud-train.conf` file. With this config file, you can adjust the number of threads used for certain training steps. 
Moreover, it is possible to adjust the number of iterations (default: 150) to achieve (slightly) better model performance.

#### Executing 'ud-train.sh'

Make sure to make the `ud-train.sh` script executable. 
On Unix-oid environments this can simply be achieved by setting the execute bit: `chmod 744 ud-train.sh`.

> [!TIP]
> As model training(s) can be a long-running task, depending on CPU type and number of CPU cores,
> the script should be started inside a [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html) instance.

Finally, execute the script via invoking `./ud-train.sh` and start brewing and enjoying some :coffee:.

The script logs each training (and evaluation) step per selected language / treebank, thus allowing progress tracking. 

#### Evaluating trained Models

After a training step succeeds, a corresponding evaluation step is executed. If you want to skip it, set `EVAL_AFTER_TRAINING` to `false`.
In case the evaluation is run, the resulting performance (accuracy) is written to files ending with `.eval`.                                                                                                                        

### Adding new Models

When adding new models to the `pom.xml`, ensure to add new models to the `expected-models.txt` file located in `opennlp-models-test`.
In addition, make sure a sha256 hash is computed on each binary artifact. 
The corresponding value must be set or updated correctly for each model type and language.                                       

## Contributing

The Apache OpenNLP project is developed by volunteers and is always looking for new contributors to work on all parts of the project. Every contribution is welcome and needed to make it better. A contribution can be anything from a small documentation typo fix to a new component.

If you would like to get involved please follow the instructions [here](https://github.com/apache/opennlp/blob/main/.github/CONTRIBUTING.md)

## Working with this fork

See the [fork guide](FORK_GUIDE.md) and [model artifact inventory](MODEL_INVENTORY.md)
for the inspected module layout, integration contracts and remaining documentation gaps.
