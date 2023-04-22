# Set environment for scala

Install Java Development Kit (JDK): Chisel is written in Scala, so JDK is required to compile and run Chisel code. Install JDK using Homebrew:

```bash
brew install openjdk
```

Install sbt: sbt is a build tool for Scala projects. You can install it using Homebrew:

```bash
brew install sbt
```

Get Chisel Template: Clone the Chisel template to your computer by running the following command:

```bash
git clone https://github.com/ucb-bar/chisel-template.git
```

Compiling and running Chisel projects: Navigate to the directory of the cloned project:

```bash
cd chisel-template
```

Then run sbt, which will start the sbt console:

```bash
sbt
```

Execute the following command in the sbt console to compile the project:

```bash
compile
```

To run the test, execute the following command:

```
test
```

Now, you can find some example files in the chisel-template project or start creating your own Chisel project. To learn how to write Chisel code, you can refer to the **[Chisel official documentation](https://www.chisel-lang.org/)**.

# What is sbt (Simple Build Tool）？

**`sbt`** (Simple Build Tool) is a build tool for Scala projects that simplifies the build process while providing powerful functionality. Here are some basic rules and commonly used commands for sbt:

1. Project structure: sbt has a predefined project structure that needs to follow the following conventions:
    - **`src/main/scala`**: source code goes here
    - **`src/main/resources`**: non-code resource files (like configuration files) go here
    - **`src/test/scala`**: test code goes here
    - **`src/test/resources`**: test resource files go here
    - **`target`**: output directory of the compilation
    - **`build.sbt`**: sbt build definition file
2. Build definition file (**`build.sbt`**): this file contains the project's build definition and configuration information. Here are some basic settings:
    - **`name`**: project name
    - **`version`**: project version
    - **`scalaVersion`**: Scala version used in the project
    - **`libraryDependencies`**: external libraries that the project depends on
    
    For example:
    
    ```bash
    name := "MyProject"
    version := "1.0"
    scalaVersion := "2.13.6"
    libraryDependencies += "org.typelevel" %% "cats-core" % "2.6.1"
    ```
    
3. Common commands:
- **`compile`**: compile the project
- **`test`**: run tests
- **`run`**: run the project
- **`clean`**: clean the compilation output and temporary files
- **`reload`**: reload the build definition file (**`build.sbt`**)
- **`update`**: update dependencies
- **`console`**: start an interactive Scala REPL with project classes and dependencies
- **`package`**: package the project as a JAR file
- **`publish`**: publish the project to a remote repository
1. Multi-module projects: sbt supports multi-module projects, allowing multiple sub-projects to be defined within a single project. In the **`build.sbt`** file, sub-projects can be defined using **`lazy val`** and each sub-project can be specified with its own settings. For example:

```bash
lazy val core = (project in file("core"))
  .settings(
    name := "MyProject-Core",
    libraryDependencies += "com.typesafe.akka" %% "akka-actor" % "2.6.16"
  )

lazy val webapp = (project in file("webapp"))
  .settings(
    name := "MyProject-WebApp",
    libraryDependencies += "com.typesafe.play" %% "play" % "2.8.11"
  )
```

In this example, we define two sub-projects named **`core`** and **`webapp`**. Each sub-project has its own dependencies

# How to start your chisel coding

### Create a repository from the template

This repository is a Github template. You can create your own repository from it by clicking the green `Use this template` in the top right. Please leave `Include all branches` **unchecked**; checking it will pollute the history of your new repository. For more information, see ["Creating a repository from a template"](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template).

### Wait for the template cleanup workflow to complete

After using the template to create your own blank project, please wait a minute or two for the `Template cleanup` workflow to run which will removes some template-specific stuff from the repository (like the LICENSE). Refresh the repository page in your browser until you see a 2nd commit by `actions-user` titled `Template cleanup`.

### Clone your repository

Once you have created a repository from this template and the `Template cleanup` workflow has completed, you can click the green button to get a link for cloning your repository. Note that it is easiest to push to a repository if you set up SSH with Github, please see the [related documentation](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh). SSH is required for pushing to a Github repository when using two-factor authentication.

`git clone git@github.com:rolyneko/test2_chisel.git
cd test2_chisel`

### Set project organization and name in build.sbt

The cleanup workflow will have attempted to provide sensible defaults for `ThisBuild / organization` and `name` in the `build.sbt`. Feel free to use your text editor of choice to change them as you see fit.

### Clean up the README.md file

Again, use you editor of choice to make the README specific to your project.

### Add a LICENSE file

It is important to have a LICENSE for open source (or closed source) code. This template repository has the Unlicense in order to allow users to add any license they want to derivative code. The Unlicense is stripped when creating a repository from this template so that users do not accidentally unlicense their own work.

For more information about a license, check out the [Github Docs](https://docs.github.com/en/free-pro-team@latest/github/building-a-strong-community/adding-a-license-to-a-repository).

### Commit your changes

`git commit -m 'Starting test2_chisel'
git push origin main`

### Did it work?

You should now have a working Chisel3 project.

You can run the included test with:

`sbt test`

You should see a whole bunch of output that ends with something like the following lines

`[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 5 s, completed Dec 16, 2020 12:18:44 PM`

If you see the above then...

### It worked!

You are ready to go.

# A simple example in chisel

This code is for computing the most common divisor

```scala
package gcd

import chisel3._
import chisel3.util.Decoupled

class GcdInputBundle(val w: Int) extends Bundle {
  val value1 = UInt(w.W)
  val value2 = UInt(w.W)
}

class GcdOutputBundle(val w: Int) extends Bundle {
  val value1 = UInt(w.W)
  val value2 = UInt(w.W)
  val gcd    = UInt(w.W)
}

/**
  * Compute Gcd using subtraction method.
  * Subtracts the smaller from the larger until register y is zero.
  * value input register x is then the Gcd.
  * Unless first input is zero then the Gcd is y.
  * Can handle stalls on the producer or consumer side
  */
class DecoupledGcd(width: Int) extends Module {
  val input = IO(Flipped(Decoupled(new GcdInputBundle(width))))
  val output = IO(Decoupled(new GcdOutputBundle(width)))

  val xInitial    = Reg(UInt())
  val yInitial    = Reg(UInt())
  val x           = Reg(UInt())
  val y           = Reg(UInt())
  val busy        = RegInit(false.B)
  val resultValid = RegInit(false.B)

  input.ready := ! busy
  output.valid := resultValid
  output.bits := DontCare

  when(busy)  {
    when(x > y) {
      x := x - y
    }.otherwise {
      y := y - x
    }
    when(x === 0.U || y === 0.U) {
      when(x === 0.U) {
        output.bits.gcd := y
      }.otherwise {
        output.bits.gcd := x
      }

      output.bits.value1 := xInitial
      output.bits.value2 := yInitial
      resultValid := true.B

      when(output.ready && resultValid) {
        busy := false.B
        resultValid := false.B
      }
    }
  }.otherwise {
    when(input.valid) {
      val bundle = input.deq()
      x := bundle.value1
      y := bundle.value2
      xInitial := bundle.value1
      yInitial := bundle.value2
      busy := true.B
    }
  }
}
```

Below is is to verify that the **`DecoupledGcd`**module can correctly compute the greatest common divisor for various input data, and it also tests the behavior of the module during input and output stalls.

```scala
package gcd

import chisel3._
import chiseltest._
import org.scalatest.freespec.AnyFreeSpec
import chisel3.experimental.BundleLiterals._

class GCDSpec extends AnyFreeSpec with ChiselScalatestTester {

  "Gcd should calculate proper greatest common denominator" in {
    test(new DecoupledGcd(16)) { dut =>
      dut.input.initSource()
      dut.input.setSourceClock(dut.clock)
      dut.output.initSink()
      dut.output.setSinkClock(dut.clock)

      val testValues = for { x <- 0 to 10; y <- 0 to 10} yield (x, y)
      val inputSeq = testValues.map { case (x, y) => (new GcdInputBundle(16)).Lit(_.value1 -> x.U, _.value2 -> y.U) }
      val resultSeq = testValues.map { case (x, y) =>
        (new GcdOutputBundle(16)).Lit(_.value1 -> x.U, _.value2 -> y.U, _.gcd -> BigInt(x).gcd(BigInt(y)).U)
      }

      fork {
        // push inputs into the calculator, stall for 11 cycles one third of the way
        val (seq1, seq2) = inputSeq.splitAt(resultSeq.length / 3)
        dut.input.enqueueSeq(seq1)
        dut.clock.step(11)
        dut.input.enqueueSeq(seq2)
      }.fork {
        // retrieve computations from the calculator, stall for 10 cycles one half of the way
        val (seq1, seq2) = resultSeq.splitAt(resultSeq.length / 2)
        dut.output.expectDequeueSeq(seq1)
        dut.clock.step(10)
        dut.output.expectDequeueSeq(seq2)
      }.join()

    }
  }
}
```

The test result shows as below.

```bash
rolycowneko@macbook-pro-13 test2_chisel % sbt test
[info] welcome to sbt 1.8.0 (Homebrew Java 20)
[info] loading settings for project test2_chisel-build from plugins.sbt ...
[info] loading project definition from /Users/rolycowneko/test2_chisel/project
[info] loading settings for project root from build.sbt ...
[info] set current project to test2_chisel (in build file:/Users/rolycowneko/test2_chisel/)
[info] GCDSpec:
[info] - Gcd should calculate proper greatest common denominator
[info] Run completed in 1 second, 611 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
```