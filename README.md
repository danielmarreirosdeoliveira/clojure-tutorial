# Clojure

As soon as you have installed **Clojure**, you can start a **repl** (read-eval-print-loop) 
session.

    $ clj
    user=> (+ 1 1)
    2

You also can put code into a file and execute it from the command line.

    ./examples/hello.clj:
    (prn (+ 1 1))

    examples$ clj hello.clj
    2

As soon as you want to distribute code across multiple source files, 
you can very easily set up projects, following very limited conventions. 

    ./examples/greeter-1/src/greeter.clj:
    (ns greeter)
    (defn greet [name] 
      (prn (str "Hello, " name "!")))

    ./examples/greeter-1/src/hello.clj:
    (ns hello
      (:require [greeter :refer :all]))
    (defn -main [& args]
      (greet (first args)))

    examples/greeter-1$ clj -m hello Daniel
    "Hello, Daniel!"

Adhering to these conventions one can access the application from the repl.

    greeter-1$ clj
    Clojure 1.9.0
    user=> (require '[greeter :refer :all])
    nil
    user=> (greet "Daniel")
    "Hello, Daniel!"
    nil

The first build tool you should check out is **deps.edn**. 
It comes as part of the language and allows you to use dependencies, 
from maven, from github, as well
as on the local file system, such that the files from the last 
example could also be laid out as follows:

    ./examples/greeter-2/application/deps.edn:
    {:deps
     {greeter {:local/root "../library"}}}

    ./examples/greeter-2/library/deps.edn:
    {}

    ./examples/greeter-2/library/src/greeter.clj:
    (ns greeter)
    (defn greet [name] 
      (prn (str "Hello, " name "!")))

    ./examples/greeter-2/application/src/hello.clj:
    (ns hello
      (:require [greeter :refer :all]))
    (defn -main [& args]
      (greet (first args)))

    examples/greeter-2/application$ clj -m hello Daniel
    "Hello, Daniel!"

Using it you can install a test runner, which 
facilitates writing unit tests with **clojure.test**, 
which is also part of the language.


    ./examples/adder/deps.edn:
    {
      :deps {com.cognitect/test-runner {
        :git/url "https://github.com/cognitect-labs/test-runner.git"
        :sha "209b64504cb3bd3b99ecfec7937b358a879f55c1"}}
      :aliases {:test {:extra-paths ["test"]
                       :main-opts ["-m" "cognitect.test-runner"]}}
    }

    ./examples/adder/src/adder.clj:
    (ns adder)
    (defn add [a b] (+ a b))

    ./examples/adder/test/adder_test.clj:
    (ns adder-test
      (:require [clojure.test :refer :all]
                [adder :refer :all]))
    (deftest test-adder
      (is (= 2 (add 1 1))))

    examples/adder$ clj -Atest
    Running tests in #{"test"}

    Testing adder-test
    
    Ran 1 tests containing 1 assertions.
    0 failures, 0 errors.

A more powerful build tool is **Leiningen** (or **lein** for short). The greeter example
looks like this, using the opportunity to show how Java can be mixed in when using Leiningen.

    ./examples/greeter-3/src/clj/hello.clj:
    (ns hello 
      (:import Greeter))
    (defn -main [& args]
      (Greeter/greet (first args)))

    ./examples/greeter-3/src/java/Greeter.java:
    public class Greeter {
        public static void greet(String name) {
            System.out.println("Hello, " + name + "!");
        }
    }

    ./examples/greeter-3/project.clj:
    (defproject greeter-3 "0.1.0-SNAPSHOT"
    :dependencies [[org.clojure/clojure "1.10.0"]]
    :main ^:skip-aot hello
    :source-paths      ["src/clj"]
    :java-source-paths ["src/java"])

    examples/greeter-3$ lein run Daniel
    Hello, Daniel!