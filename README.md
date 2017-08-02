# Modularity workshop

**This is still a work in progress.**

This workshop should teach you how you can create your own modules.

The plan is to utilize existing guides and just link to them.

Before we start, feel free to look into [Stephen Gallagher's](https://github.com/sgallagher) blog posts on module packaging:
 * [Sausage Factory: An introduction to building modules in Fedora](https://sgallagh.wordpress.com/2017/05/30/sausage-factory-an-introduction-to-building-modules-in-fedora/)
 * [Sausage Factory: Advanced module building in Fedora](https://sgallagh.wordpress.com/2017/06/30/sausage-factory-advanced-module-building-in-fedora/)


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

Modulemd stands for two things:

 1. a specification
 2. a library


### The specification

The modulemd file allows you to describe your module:
 * module description
 * licensing
 * build-time and runtime dependencies
 * module API
 * module components and artifacts

Modulemd is agnostic of packaging formats (at the same time, it's implemnted for RPM only now).

 * [the specification](https://pagure.io/modulemd/blob/master/f/spec.yaml)
 * [a real example](http://pkgs.fedoraproject.org/cgit/modules/nodejs.git/tree/nodejs.yaml?h=f26)
 * [list of existing modules](http://pkgs.fedoraproject.org/cgit/modules)
 * interesting modules:
   * platform module
     * TODO: link; platform should have builds available during first week of August, 2017
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

It will get us this modulemd:

```
data:
  api:
    rpms: [nodejs, npm]
  components:
    rpms:
      http-parser: {rationale: Build and runtime dependency.}
      nodejs: {buildorder: 10, rationale: Runtime dependency.}
  dependencies:
    buildrequires: {base-runtime: f26, shared-userspace: f26}
    requires: {base-runtime: f26, shared-userspace: f26}
  description: ''
  license:
    module: [MIT]
  summary: ''
document: modulemd
version: 1
```


### Figuring out dependencies

[depchase](https://github.com/fedora-modularity/depchase)

```
$ ./depchase -a x86_64 -c ./repos.cfg.sample resolve nodejs npm
http-parser-2.7.1-5.fc26.x86_64
compat-openssl10-1:1.0.2j-6.fc26.x86_64
pcre-8.41-1.fc27.x86_64
p11-kit-trust-0.23.7-1.fc27.x86_64
sed-4.4-1.fc26.x86_64
libicu-57.1-4.fc26.x86_64
libtasn1-4.12-1.fc27.x86_64
grep-3.1-1.fc27.x86_64
ca-certificates-2017.2.16-2.fc27.noarch
libuv-1:1.13.1-1.fc27.x86_64
filesystem-3.3-1.fc27.x86_64
libacl-2.2.52-16.fc27.x86_64
tzdata-2017b-1.fc27.noarch
libatomic_ops-7.4.6-1.fc27.x86_64
fedora-release-27-0.2.noarch
make-1:4.2.1-2.fc26.x86_64
libattr-2.4.47-19.fc27.x86_64
fedora-repos-27-0.2.noarch
fedora-repos-rawhide-27-0.2.noarch
npm-1:5.3.0-1.8.2.1.1.fc27.x86_64
coreutils-8.27-13.fc27.x86_64
...
```


## `mbs-build`

[Link](https://pagure.io/fm-orchestrator/)

[Local builds](https://docs.pagure.org/modularity/development/building-modules/building-local.html)  
[Infrastructure builds](https://docs.pagure.org/modularity/development/building-modules/building-infra.html)


## Testing

~~Modularity testing framework~~ [Meta test family](https://github.com/fedora-modularity/meta-test-family)

[MTF Guide](http://meta-test-family.readthedocs.io/en/latest/user_guide/index.html)


## We need your feedback!
