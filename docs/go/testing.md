Testing
=======

## Database layer testing

Every application has Store/Database layer which mostly consists of CRUD operations. There are some tips on how to
write those test and what to test.

**Quick tips:**

- Don't mock your database. Use [dockertest](https://github.com/ory/dockertest).
- Test your migrations.
- Run your CRUD operations more than once, run 1000 times.
- Use fake and random data. Library such as [gaofakeit](https://github.com/brianvoe/gofakeit) helps you with that.

Let's discuss every point mentioned above.

**Mocks**

You are testing your database layer, which means you are testing your implementation and not some abstract layer.
Mocks are far away from a real Postgres or MySQL database. Testing against real database helps you to reveal some
bugs before deploying your application to your real environment. Creating a new docker image for testing takes a few
seconds, so don't be afraid of it.

**Migrations**

If your migrations are shipped with an application, test the migrations by calling `up` and `down` twice. It will
help you avoid silly mistakes.

**1000 times**

During the tests create/read/update/delete more than once to catch deadlocks, avoid leaky resources and find some
hidden bugs. For example, the method that should return only one record, returns more than one, or deleting one
record in reality deletes more than one.

**Fake data**

Imagine that you mistakenly used `int8` in your database, but you are trying to save `int32` instead. If you use a
constant value during the tests, you probably won't find the issue. Therefore, using some random data might help
you to discover those problems during the tests.

## Event-driven applications testing

You definitely have a part of some application that listens for some events incoming from a message broker and
writes them to the database. Our tests should cover a few things:

- All the valid events were received successfully.
- All the data from events was mapped correctly and saved to DB.
- Invalid data was rejected and moved somewhere else.

**Counter**

Don't use `time.Sleep` and wait for a constant time until all the events were digested. On slow machines it might
take longer and the test will fail. On faster machines you are wasting time. I suggest to have a custom counter
injected into your `Subscriber/Listener` object, and read it every few seconds until all the event were drained.
After that fetch all the saved records from the database and check for equality with sent events. This strategy also
helps to count the number of invalid events.

**Containers**

Use `dockertest` or similar library to spin up the containers programmatically during the tests. Don't try to mock
`kafka` or any other message broker. Creating `kafka` container for tests might be very slow and consume a lot of
CPU and RAM, so I would suggest to use something which is Kafka API compatible,
e.g. [redpanda](https://github.com/redpanda-data/redpanda/).
The latter doesn't need zookeeper, and it starts much faster than kafka.
