# Modularity workshop

**This is still a work in progress.**

This workshop should teach you how you can create your own modules.

The plan is to utilize existing guides and just link to them.


## High-level agenda

1. [1-minute-intro to modularity](#1-minute-intro)
2. [modulemd](#modulemd)
3. [Workflow](#workflow)
4. [`mbs-build`](#mbs-build)
5. [Testing](#testing)
7. [Feedback](#we-need-your-feedback)
8. Beer


# The workshop


## 1-minute-intro

 * decouple component version from an [OS release](https://docs.pagure.org/modularity/)
 * provide a solution for component maintainers to supply more versions of one component
 * lifecycles and support level can vary between modules
 * module authors to define how the module is meant to be built and utilized by users


## modulemd

 * a specification
 * a library


### The specification

 * [link](https://pagure.io/modulemd/blob/master/f/spec.yaml)
 * [a real example](http://pkgs.fedoraproject.org/cgit/modules/nodejs.git/tree/nodejs.yaml?h=f26)
 * [list of modules](http://pkgs.fedoraproject.org/cgit/modules)
 * interesting modules:
   * platform module
     * to have builds available during first week of August, 2017
     * is still a work in-progress
   * base-runtime
   * shared-userspace
   * common-build-dependencies
   * common-build-dependencies-bootstrap
 * runtime dependencies
 * build-time dependencies
 * module components
 * filtering
 * API


## Workflow

[Guide](https://docs.pagure.org/modularity/development/building-modules.html)

 * guidelines (TODO)
 * [branching](https://fedoraproject.org/wiki/Changes/ArbitraryBranching#Current_status)
 * bootstrapping

### Modularity tools

[Link](https://pagure.io/modularity/modularity-tools)

```
$ mod-tools rpm2module nodejs npm
```


### Figuring out dependencies

[Link](https://github.com/fedora-modularity/depchase)


## `mbs-build`

[Link](https://pagure.io/fm-orchestrator/)

[Local builds](https://docs.pagure.org/modularity/development/building-modules/building-local.html)
[Infrastructure builds](https://docs.pagure.org/modularity/development/building-modules/building-infra.html)


## Testing

~~Modularity testing framework~~ [Meta test family](https://github.com/fedora-modularity/meta-test-family)

[MTF Guide](http://meta-test-family.readthedocs.io/en/latest/user_guide/index.html)


## We need your feedback!
