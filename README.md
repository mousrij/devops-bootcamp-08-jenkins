# DevOps Bootcamp
## Module 08 "Build Automation & CI/CD with Jenkins"
<br />

This project contains notes related to the videos and exercises in Module 08 "Build Automation & CI/CD with Jenkins" of Nana Janashia's [DevOps Bootcamp](https://www.techworld-with-nana.com/devops-bootcamp).

### Exercises
It also contains the NodeJS application the exercises of this module are based on.

#### Test
The project uses jest library for tests (see "test" script in package.json).
There is one test (server.test.js) in the project that checks whether the main index.html file exists in the project. 

To run the nodejs test, execute:

    npm run test

Make sure to download jest library before running test, otherwise jest command defined in package.json won't be found:

    npm install

In order to see failing test, remove index.html or rename it and run tests.

### Demo Projects
And finally it contains the following demo projects:
