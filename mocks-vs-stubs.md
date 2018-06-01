# Testing fun!
This document briefly explains the differences between the following testing methods:
- Using Mocks
- Using Stubs
- Using Functional Interfaces (available in Java 8)

To demonstrate the differences, I'm going to re-use [the code examples written by Pam](https://git.realestate.com.au/financial-experiences/financial-profile-api/tree/tech/mocks-vs-no-mocks).

Basically we have a `S3Client` class that has a `upload()` method that we want to test. The `S3Client` class takes a `S3Writer` and a `S3Reader` in its constructor and we specificially want to test whether the `s3Writer.putObject` is being called with expected parameters.

The `S3Client` class:
```
@Component
public class S3Client {

    private S3Writer s3Writer;
    private S3Reader s3Reader;

    @Autowired
    S3Client(S3Writer writer, S3Reader reader) {
        this.s3Writer = writer;
        this.s3Reader = reader;
    }

    public PutObjectResult upload(String bucket, String key, Object object) throws AmazonServiceException {
        try {
            //stuff

            PutObjectRequest objectToSave = new PutObjectRequest(bucket, key, stream, metadata);

            return s3Writer.putObject(objectToSave); //<- Test that objectToSave has the expected data.

        } catch (Exception e) {
            throw new Exception();
        }
    }
}
```
When we test this method, we don't want to use a **_real_** s3 bucket or set up **_real_** AWS credentials. So there are 3 ways we could go about this.

## What's the difference between Mocks and Stubs?
Firstly, instead of using the **_real_** `s3Writer` we can create a fake s3Writer. The fake s3Writer can have new behaviour for `upload()`, so that instead of uploading data to a **_real_** s3 bucket, we can tell it to do some checks and maybe return something useful which we can assert on.

> How this `fakeS3Writer` is used is the key factor in determining whether we are mocking or stubbing out the s3Writer.

If we use `fakeS3Writer` to verify behaviours, for example if we did `fakeS3Writer.call.putObject()`, the `fakeS3Writer` is a mock of the `s3Writer`.

If we use `fakeS3Writer` to verify state, for example `fakeS3Writer.getObjectToSave.getBucketName() == "test.bucket"`, then `fakeS3Writer` is a stub of the `s3Writer`.

Read more on the two here: [Martin Fowler - Mocks arn't stubs](https://martinfowler.com/articles/mocksArentStubs.html)

So, the main difference between the two is that mocks are used to verify *behaviours* and stubs are used to verify *state*. Depending on how we choose to test our code it can significantly change our design.

So which one to use?

## Using Stubs
Below is an implementation of a `fakeS3Writer`, where on `putObject()` takes the `PutObjectRequest` parameter and stores it on a private attribute, which can be accessed by `getRequestPassed()`. The reason for this is so that the request parameter passed in can be later checked in a test for correctness.

```
 private class FakeS3Writer implements S3Writer {
    private PutObjectRequest requestPassed;

    @Override
    public PutObjectResult putObject(PutObjectRequest request) {
        requestPassed = new PutObjectRequest(request.getBucketName(), request.getKey(), request.getFile());
        requestPassed.setMetadata(request.getMetadata());
        return new PutObjectResult();
    }

    public PutObjectRequest getRequestPassed() {
        return requestPassed;
    }
}
```
And our test could look like this:
```
@Test
public void it_should_upload_with_content_type_json_with_stub_class() {

    FakeS3Writer fakeWriter = new FakeS3Writer();

    // instantiate s3Client with our fakeWriter instead of a real S3Writer.
    S3Client s3Client = new S3Client(fakeWriter, reader);

    // method under test which calls s3Writer.putObject() using our fakeWriter
    s3Client.upload("test.bucket", "fake/path", map);

    // access private attribute on our fakeS3Writer
    PutObjectRequest requestPassedIn = fakeWriter.getRequestPassed();

    // assertions
    assertThat(requestPassedIn.getBucketName(), is("test.bucket"));
    assertThat(requestPassedIn.getKey(), is("fake/path"));
}
```
__Stub considerations:__
- no libraries used
- our assertions are explicit
- if within the `s3Client.upload()` a `s3writer.putObject()` is not found, this test will correctly fail.
- code can be verbose

__Design considerations:__
- Super easy being able to pass a `fakeWriter` to the `s3Client` class because the code was nicely designed to be able to receive it as a param.
- The `fakes3Writer` is now actually being used as some sort of a `S3ParamChecker`, which can make us think about what we are actually testing.


## Using Functional Interfaces (available in Java 8)
In Java 8 functional interfaces provide a way to replace objects with functions. This means that instead of using a `fakeS3Writer` class that we use, we can pass in a function that does the checking for us instead.

To do this, we first mark the `S3Writer` to be a `@FunctionalInterface` - we can ONLY do this if the class has **1 method**.
```
@FunctionalInterface
public interface S3Writer {
    PutObjectResult putObject(PutObjectRequest request);
}
```
We then create a function that can replace the `S3Writer` with our checks.
```
private PutObjectResult s3WriterParamCheck(PutObjectRequest request) {

    assertThat(request.getBucketName(), is("test.bucket"));
    assertThat(request.getKey(), is("fake/path"));

    return new PutObjectResult();
}
```

Our test code could then transform into this
```
@Test
public void it_should_upload_with_content_type_json_with_functional_interfaces() {

    S3Client s3Client = new S3Client( (request) -> { return s3WriterParamCheck(request); },
                                       reader);

    s3Client.upload("test.bucket", "fake/path", map);
}
```
We could further simplify this by nesting the function within
```
@Test
public void it_should_upload_with_content_type_json_with_functional_interfaces() {

    S3Client s3Client = new S3Client( (request) -> {
                                        assertThat(request.getBucketName(), is("test.bucket"));
                                        assertThat(request.getKey(), is("fake/path"));

                                        return new PutObjectResult();
                                       },
                                       reader);

    s3Client.upload("test.bucket", "fake/path", map);
}
```
__Pretty neat right!__

The `S3Client` still takes 2 arguments into its constructor, however, the writer argument is replaced with a function. When the writer is used further down in the chain (within `s3Client.upload()`) it'll execute our function parameter we passed in!

__Functional interfaces considerations:__
- smarter design and less code
- there’s only one function! we don’t have to define the types for the parameters
- can pass in a function that throws an exception to test the fail case
- no mocking libraries used
- our assertions are explicit

__Design considerations__
- if within the `s3Client.upload()` method a `s3writer` is not used, the tests will not fail. This will prompt us to write a seperate test to catch this case. Probably a good thing.
- We have to change our design to use an interface. Interfaces can be really good. This video talks about it [around the 24min mark on why](https://www.youtube.com/watch?v=VDfX44fZoMc&t=2655s).

## Using Mocks
Mocks can be really easy, especially if we use libraries.

Below we mock the `AmazonS3Client` class and catch for any calls that hit `putObject()`. We set up all the dependencies that are required for us to test our subject.

```
@Test
public void it_should_upload_with_content_type_json_with_a_mocking_library() {

    AmazonS3 mockS3 = mock(AmazonS3Client.class);

    when(mockS3.putObject(anyObject())).thenReturn(new PutObjectResult());

    S3WriterImpl fakeWriter = new S3WriterImpl(mockS3);

    S3Client s3Client = new S3Client(fakeWriter, reader);

    s3Client.upload("test.bucket", "fake/path", map);

    ArgumentCaptor<PutObjectRequest> argumentCaptor = ArgumentCaptor.forClass(PutObjectRequest.class);

    verify(mockS3, times(1)).putObject(argumentCaptor.capture()); // verify that we've called putObject once

    PutObjectRequest requestPassedIn = argumentCaptor.getAllValues().get(0);

    assertThat(requestPassedIn.getBucketName(), is("test.bucket")); // we can still assert on parameters using the mock
    assertThat(requestPassedIn.getKey(), is("fake/path"));
}
```

__Mock considerations:__
- most of the time using a library which we have to import
- very easy and fast to set up
- we would mock all dependencies required

__Design considerations__
- By using a mock, we never consider introducing an interface into our design.
- It's easy to not think about the design and dependencies if they are easy to mock.
- Just testing if something has been called may not be a clear test, so we have to set up an argumentCaptor to assert on the correctness of params.



## Resources
- [Understanding mock objects video](https://www.youtube.com/watch?v=fAb_OnooCsQ&t=408s)
- [Mocks and Stubs video](https://www.youtube.com/watch?v=EaxDl5NPuCA)
- [Boundaries video](https://www.youtube.com/watch?v=eOYal8elnZk)
