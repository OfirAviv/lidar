# LeaderLidar
===========

LeaderLidar is a car safety vendor. Their primary product, LeaderLidar, is comprised of two packages:
- telemetry - a package that integrates to car metrics and formats them into an internal events protocol.
- analytics - a package that accepts events and decides on actions such as "beep" or "brake".

The company releases their product as a zip file.

You were recruited to help automate the company CI and Release processes. Following are your tasks:

1. Setup a CI environment in AWS
   - Gitlab. The company has 4 repositories
     - telemetry
     - analytics
     - product
     - testing
   - Artifactory
   - Jenkins
2. Make sure that all 4 repos work with artifactory by:
   - Set remote artifact repositories either via their `pom.xml`s or via `settings.xml` (see https://maven.apache.org/guides/introduction/introduction-to-repositories.html).
   - Testing `mvn deploy`.
   - If you fail on permissions, make sure you configure `.m2/settings.xml` with appropriate credentials.
3. Create an MBP that performs both CI and Release according to the instructions below.

Continuous Integration (CI)
---------------------------
telemetry:
- Feature branches ("feature/...") should pass build and unit test for every commit (mvn package). If the commit message contains #e2e, End to end tests will also be executed using analytics:99-SNAPSHOT(see below).
- Master always performs build, e2e tests and publish to artifactory (`mvn deploy -DskipTests`).
- Release branches ("release/...") attempt a release (see below).
analytics: same logic
product:
- Only release branches are CIed.
testing:
- master only. every commit performs build, e2e tests and publish to artifactory (`mvn deploy -DskipTests`).

Release
-------
- Versions `x.y.z` come from branch `release/x.y`:
  - Fix version in pom to `x.y.z` (`mvn versions:set -DnewVersion=${YOUR_VERSION}`).
  - If there are any other `com.lidar` dependencies, make sure they are on the same `x.y` with latest `z` (use `mvn dependency:list`).
  - Build and unit test (`mvn package`).
  - analytics & product perform E2E tests. telemetry does not.
  - E2E test using:
    - For analytics `x.y.z` use latest telemetry `x.y`.
    - For product use the analytics and telemetry in the product zip.
  - If all goes well:
    - Publish to artifactory (`mvn deploy -DskipTests`).
    - `git tag x.y.z`

E2E Testing
-----------
- Fetch the jars you want to test:
  - for the product, you can find them in the zip.
  - in analytics you can find one in the target folder and the other can be downloaded from artifactory using `curl`.
- To execute testing you need to run simulator, making sure that simulator, telemetry and analytics jars are in its classpath (`java -cp <jar1>:<jar2>:<jar3> com.lidar.simulation.Simulator`).
- simulator will run all tests found in `tests.txt` on its execution folder (you can use `tests-sanity.txt` initially, but the goal is to run `tests-full.txt`, and fast).
- simulator returns non zero for failed tests.
- you may change the name of the test files provided to the filename the code uses.
