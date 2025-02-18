# konserve-s3

A [S3](https://aws.amazon.com/s3) backend for [konserve](https://github.com/replikativ/konserve). 


## Usage

Add to your dependencies:

[![Clojars Project](http://clojars.org/io.replikativ/konserve-s3/latest-version.svg)](http://clojars.org/io.replikativ/konserve-s3)

### Synchronous Execution

``` clojure
(require '[konserve-s3.core :refer [connect-s3-store]]
         '[konserve.core :as k])

(def s3-spec
  {:region "us-west-1"
   :bucket "konserve-demo"})

(def store (connect-s3-store s3-spec :opts {:sync? true}))

(k/assoc-in store ["foo" :bar] {:foo "baz"} {:sync? true})
(k/get-in store ["foo"] nil {:sync? true})
(k/exists? store "foo" {:sync? true})

(k/assoc-in store [:bar] 42 {:sync? true})
(k/update-in store [:bar] inc {:sync? true})
(k/get-in store [:bar] nil {:sync? true})
(k/dissoc store :bar {:sync? true})

(k/append store :error-log {:type :horrible} {:sync? true})
(k/log store :error-log {:sync? true})

(let [ba (byte-array (* 10 1024 1024) (byte 42))]
  (time (k/bassoc store "banana" ba {:sync? true})))

(k/bassoc store :binbar (byte-array (range 10)) {:sync? true})
(k/bget store :binbar (fn [{:keys [input-stream]}]
                               (map byte (slurp input-stream)))
       {:sync? true})

```

### Asynchronous Execution

``` clojure
(ns test-db
  (require '[konserve-s3.core :refer [connect-store]]
           '[clojure.core.async :refer [<!]]
           '[konserve.core :as k])

(def s3-spec
  {:region "us-west-1"
   :bucket "konserve-demo"})

(def store (<! (connect-store s3-spec :opts {:sync? false})))

(<! (k/assoc-in store ["foo" :bar] {:foo "baz"}))
(<! (k/get-in store ["foo"]))
(<! (k/exists? store "foo"))

(<! (k/assoc-in store [:bar] 42))
(<! (k/update-in store [:bar] inc))
(<! (k/get-in store [:bar]))
(<! (k/dissoc store :bar))

(<! (k/append store :error-log {:type :horrible}))
(<! (k/log store :error-log))

(<! (k/bassoc store :binbar (byte-array (range 10)) {:sync? false}))
(<! (k/bget store :binbar (fn [{:keys [input-stream]}]
                            (map byte (slurp input-stream)))
            {:sync? false}))
```

Note that you do not need full S3 rights if you manage the bucket outside, i.e.
create it before and delete it after usage form a privileged account. Connection
will otherwise create a bucket and it can be recursively deleted by
`delete-store.` You can activate [Amazon X-Ray](https://aws.amazon.com/xray/) by
setting `:x-ray?` to `true` in the S3 spec.

## Authentication

A [common
approach](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)
to manage AWS credentials is to put them into the environment variables as
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` to avoid storing them in plain
text or code files. Alternatively you can provide the credentials in the
`s3-spec` as `:access-key` and `:secret`.

## Commercial support

We are happy to provide commercial support with
[lambdaforge](https://lambdaforge.io). If you are interested in a particular
feature, please let us know.

## License

Copyright © 2023 Christian Weilbach

Licensed under Eclipse Public License (see [LICENSE](LICENSE)).
