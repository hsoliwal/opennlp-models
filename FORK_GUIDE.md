<!-- SPDX-License-Identifier: Apache-2.0 -->

# Fork guide: model packaging, training and validation

This public fork of [apache/opennlp-models](https://github.com/apache/opennlp-models) is documented at [63c94ea855e7](https://github.com/hsoliwal/opennlp-models/tree/63c94ea855e78f68fc0e07fd99c72a966b682d54), inspected 2026-10-14. It packages NLP model resources as Maven artifacts. Inference algorithms and runtime APIs live in OpenNLP core.

Start with the [README](README.md), [complete artifact inventory](MODEL_INVENTORY.md), [contribution guide](.github/CONTRIBUTING.md), [LICENSE](LICENSE) and [NOTICE](NOTICE).

## Project and module map

The untruncated tree contains 153 POM projects: one root and 152 descendants. Every POM was inspected. There is one Java source file in one namespace; most modules package resources.

| Project/module | Responsibility and contract |
| --- | --- |
| [Root POM](pom.xml) | Aggregates seven direct children; supplies versions, prerequisites, download/checksum policy and resource attachment. A build POM, not an inference library |
| [Language detector](opennlp-models-langdetect/pom.xml) | One model artifact: langdetect-183.bin, language root, model version 1.8.3; its download URL includes a version directory |
| [Lemmatizer family](opennlp-models-lemmatizer/pom.xml) | POM aggregator of 36 language-specific lemma model JARs, consumed by separately supplied runtime lemmatizer APIs |
| [POS family](opennlp-models-pos/pom.xml) | POM aggregator of 36 part-of-speech model JARs, consumed by a runtime tagger with tokenized input |
| [Sentence detector family](opennlp-models-sentdetect/pom.xml) | POM aggregator of 36 sentence-boundary model JARs |
| [Tokenizer family](opennlp-models-tokenizer/pom.xml) | POM aggregator of 36 token-boundary model JARs |
| [Validation module](opennlp-models-test/pom.xml) | Build-time Java entry point with all model dependencies and a verify-phase exec:java binding; examines packaging, not linguistic accuracy |
| [Training parent](opennlp-models-training/pom.xml) | POM aggregator of the UD resource module |
| [UD training resources](opennlp-models-training/opennlp-models-training-ud/pom.xml) | Packages a shell script and configuration; execution uses an external OpenNLP CLI and locally supplied Universal Dependencies treebanks |

All 145 concrete model artifacts are individually linked in [MODEL_INVENTORY.md](MODEL_INVENTORY.md). Parent projects and concrete distribution packages have different roles; neither a directory name nor a parent POM establishes a runtime API.

## Artifact consumption and resource lifecycle

Choose a concrete family/language artifact. For example, [English tokenization](opennlp-models-tokenizer/opennlp-models-tokenizer-en/pom.xml) declares opennlp-models-tokenizer-en and opennlp-en-ud-ewt-tokens-1.3-2.5.4.bin. Pin an available artifact version in the consuming build and separately pin the runtime that loads it.

The [root POM](pom.xml) binds downloading to generate-resources, writes binaries under each module's target/models, checks model.sha256 and attaches that resource directory. Leaf POMs filter model.properties from src/main/resources. A checkout therefore need not contain the binary. Building can require network access and writable Maven/build caches.

Inputs are the manifest's URL, filename, language, model/tool version and checksum. Outputs are resource-bearing JARs. An unavailable download, checksum mismatch and incompatible runtime/model are different failure modes. Preserve identity and checksum when recording an evaluation, rather than silently substituting a model. Packaging success establishes neither model quality nor suitability for a domain.

## Java package and API contract

The only checked-in Java namespace is [org.apache.opennlp](opennlp-models-test/src/main/java/org/apache/opennlp), owned by validation tooling. Its only type is [ModelValidator](opennlp-models-test/src/main/java/org/apache/opennlp/ModelValidator.java).

- main(String[] args) requires exactly one argument: the validation module base directory. It uses the parent as reactor root and reads expected-models.txt via the thread context classloader
- It walks the reactor for matching model JARs, excluding the validation subtree and filenames containing javadoc or source; stale build artifacts can affect its count
- It compares expected-model count with JAR count. Count mismatch or a missing expected resource raises IllegalArgumentException; I/O failures are wrapped as runtime exceptions
- isModelInJar(Path, String) returns whether a JAR entry exactly matches the expected name. It closes its JarFile via try-with-resources and does not load or evaluate the model
- Source-review caveat: main currently executes return on the first successful expected-model match, exiting the method. A successful run must therefore not be described as exhaustive content validation. This observation has not been reproduced by an executed test

The expected list is [expected-models.txt](opennlp-models-test/src/main/resources/expected-models.txt); invocation is defined in the [module POM](opennlp-models-test/pom.xml). No Java test-source tree was present. Native package documentation, detailed method parameter/error docs and a regression test for exhaustive iteration remain source-change backlog items.

## Build, validation and training

The [root POM](pom.xml) enforces Java 17+ and Maven 3.6.3+. From the repository root:

- mvn clean package: package reactor resources; stops before the validator bound to verify
- mvn clean verify: also reaches configured license and packaging-validation steps, subject to the caveat above
- mvn -pl opennlp-models-tokenizer/opennlp-models-tokenizer-en -am package: select one model and reactor prerequisites; not a full validation pass

These are manifest-derived instructions. Builds, downloads, tests, inference, validation and training were NOT RUN for this editorial change.

Training uses [ud-train.sh](opennlp-models-training/opennlp-models-training-ud/src/main/resources/ud-train.sh) and [ud-train.conf](opennlp-models-training/opennlp-models-training-ud/src/main/resources/ud-train.conf). Review the script's relative paths: this snapshot names OpenNLP 2.5.4 and UD treebanks 2.16. Tokenizer, POS, sentence and lemma training plus post-training evaluation are enabled; release preparation is disabled. Outputs are model binaries and .train/.eval logs. Training needs external corpora, compute and corpus-license review. Packaging released binaries does not retrain them.

## Proposed M3 use, not implemented

A future M3 text-processing stage could consume these versioned linguistic resources. Record artifact coordinate, model filename/hash, runtime version, language and evaluation dataset. Report loading separately from task-quality results; model output is derived evidence, not canonical source structure. No M3 adapter, semantic recognizer, proof mechanism or integration test is established by this inventory.

## Coverage and remaining work

- Structural coverage: all 153 POM projects and 145 model artifacts indexed; the sole Java package and its two public entry points reviewed
- Semantic coverage: packaging/training lifecycle and validator contract described; per-language quality and runtime compatibility remain unverified
- Native API documentation: no source files changed; missing package/method semantics remain separate work
- Verification: source/tree/POM inspection and documentation navigation checks only; no build or test pass claimed

When refreshing upstream, reconcile every POM with the expected model list. Keep structural inventory, binary-integrity validation and linguistic evaluation as separate results.
