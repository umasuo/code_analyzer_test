# Code Analyzer and Test Coverage

This repository contains team-wide configurations of tools we used to analyze code, test code and report test coverage. **Be careful and get team approval** before making any changes. 
The tools include:
* `checkstyle`
* `pmd`
* `findbugs`
* `com.palantir.jacoco-coverage` 

The `com.palantir.jacoco-coverage` is an external dependency that requires extra buildscript setup (see below). 

To use it, perform the following steps: 

## 1. Add a submodule in your repository

Add this repository as a submodule, so that you can keep your resources up to date

`git submodule add https://github.com/reactivesw/code_analyzer_test.git`

use the fellow command to update the submodule

`git submodule foreach git pull`


## 2. Cusotmize for Your Project
Add a folder named`code_analyzer_test_local` to your project root, it should contain two files:`code_analyzer.gradle`,`code_test_coverage.gradleadle` that are created from the following contents:

### 2.1 `code_analyzer.gradle`
```groovy

apply plugin: 'checkstyle'
apply plugin: 'pmd'
apply plugin: 'findbugs'

// apply all configurations
apply from:'code_analyzer_test/code_analyzer_config.gradle'

/*************checkstyle(use google java style)***************/

checkstyle{
    //exclude the package you do not want to check
    //checkstyleMain.exclude 'io/reactivesw/customerauthentication/grpc/*'
}

/*************PMD(Project Manager Design)***************/
tasks.withType(Pmd) {
    //exclude the package you do not want to check
    //exclude 'io/reactivesw/customerweb/Application.java'
}

/*************find bug***************/
tasks.withType(FindBugs) {
    //exclude the package you do not want to check
    /* findBugs doesn't work if the filter is empty. Comment all if nothing to exclude
    classes = classes.filter {
        !it.path.contains('io/reactivesw/customerauthentication/grpc/')
    }
    */
}
```

### 2.2 `code_test_coverage.gradle`
```groovy
// for spock unit test
dependencies {
    testCompile('org.springframework.boot:spring-boot-starter-test')
    testCompile('org.spockframework:spock-spring:1.0-groovy-2.4')
}

// for code coverage
apply plugin: 'com.palantir.jacoco-coverage'
apply from:'code_analyzer_test/test_coverage_config.gradle'

// config of coverage check
// see document: https://github.com/palantir/gradle-jacoco-coverage
jacocoCoverage {
    // Scopes can be exempt from all coverage requirements by exact scope name or scope name pattern.
    fileThreshold 0.0, "Application.java"
//    packageThreshold 0.0, "org/company/module"
//    fileThreshold 0.0, ~".*Test.java"
}
```

## 3. Change `build.gradle` to Apply Code Analyzer and Test Coverage

### 3.1 Add External classpath Dependency
Add `classpath 'com.palantir:jacoco-coverage:0.4.0'` in the `buildscript { dependencies {...}}` section. 
```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        // for unit test code coverage  -- need to find a way to put it in its file
        classpath 'com.palantir:jacoco-coverage:0.4.0'
    }
}
```

### 3.2 Apply Analyzer and Code Coverage

Then apply the customized configuration files to the `build.gradle` file -- better in the later section. 

```groovy 
apply from: 'code_analyzer_test_local/code_analyzer.gradle'
apply from: 'code_analyzer_test_local/code_test_coverage.gradle'
```