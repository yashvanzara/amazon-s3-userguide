# Get an object from an Amazon S3 bucket using an AWS SDK<a name="example_s3_GetObject_section"></a>

The following code examples show how to read data from an object in an Amazon S3 bucket\.

**Note**  
The source code for these examples is in the [AWS Code Examples GitHub repository](https://github.com/awsdocs/aws-doc-sdk-examples)\. Have feedback on a code example? [Create an Issue](https://github.com/awsdocs/aws-doc-sdk-examples/issues/new/choose) in the code examples repo\. 

------
#### [ \.NET ]

**AWS SDK for \.NET**  
  

```
        /// <summary>
        /// Shows how to download an object from an Amazon S3 bucket to the
        /// local computer.
        /// </summary>
        /// <param name="client">An initialized Amazon S3 client object.</param>
        /// <param name="bucketName">The name of the bucket where the object is
        /// currently stored.</param>
        /// <param name="objectName">The name of the object to download.</param>
        /// <param name="filePath">The path, including filename, where the
        /// downloaded object will be stored.</param>
        /// <returns>A boolean value indicating the success or failure of the
        /// download process.</returns>
        public static async Task<bool> DownloadObjectFromBucketAsync(
            IAmazonS3 client,
            string bucketName,
            string objectName,
            string filePath)
        {
            // Create a GetObject request
            var request = new GetObjectRequest
            {
                BucketName = bucketName,
                Key = objectName,
            };

            // Issue request and remember to dispose of the response
            using GetObjectResponse response = await client.GetObjectAsync(request);

            try
            {
                // Save object to local file
                await response.WriteResponseStreamToFileAsync($"{filePath}\\{objectName}", true, System.Threading.CancellationToken.None);
                return response.HttpStatusCode == System.Net.HttpStatusCode.OK;
            }
            catch (AmazonS3Exception ex)
            {
                Console.WriteLine($"Error saving {objectName}: {ex.Message}");
                return false;
            }
        }
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/dotnetv3/S3#code-examples)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/goto/DotNetSDKV3/s3-2006-03-01/GetObject) in *AWS SDK for \.NET API Reference*\. 

------
#### [ C\+\+ ]

**SDK for C\+\+**  
  

```
bool GetObject(const Aws::String& objectKey,
    const Aws::String& fromBucket, const Aws::String& region)
{
    Aws::Client::ClientConfiguration config;

    if (!region.empty())
    {
        config.region = region;
    }

    Aws::S3::S3Client s3_client(config);

    Aws::S3::Model::GetObjectRequest object_request;
    object_request.SetBucket(fromBucket);
    object_request.SetKey(objectKey);

    Aws::S3::Model::GetObjectOutcome get_object_outcome = 
        s3_client.GetObject(object_request);

    if (get_object_outcome.IsSuccess())
    {
        auto& retrieved_file = get_object_outcome.GetResultWithOwnership().
            GetBody();

        // Print a beginning portion of the text file.
        std::cout << "Beginning of file contents:\n";
        char file_data[255] = { 0 };
        retrieved_file.getline(file_data, 254);
        std::cout << file_data << std::endl;

        return true;
    }
    else
    {
        auto err = get_object_outcome.GetError();
        std::cout << "Error: GetObject: " <<
            err.GetExceptionName() << ": " << err.GetMessage() << std::endl;

        return false;
    }
}

int main()
{
    Aws::SDKOptions options;
    Aws::InitAPI(options);
    {
        //TODO: Change bucket_name to the name of a bucket in your account.
        const Aws::String bucket_name = "<Enter bucket name>";
        
        //TODO: The bucket "DOC-EXAMPLE-BUCKET" must have been created and previously loaded with "my-file.txt". 
        //See create_bucket.cpp and put_object.cpp to create a bucket and load an object into that bucket.
        const Aws::String object_name = "<Enter object name>";
       
        //TODO: Set to the AWS Region in which the bucket was created.
        const Aws::String region = "us-east-1";

        if (!GetObject(object_name, bucket_name, region))
        {
            return 1;
        }
    }
    Aws::ShutdownAPI(options);

    return 0;
}
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/cpp/example_code/s3#code-examples)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/goto/SdkForCpp/s3-2006-03-01/GetObject) in *AWS SDK for C\+\+ API Reference*\. 

------
#### [ Java ]

**SDK for Java 2\.x**  
Read data as a byte array\.  

```
    public static void getObjectBytes (S3Client s3, String bucketName, String keyName, String path ) {

        try {
            GetObjectRequest objectRequest = GetObjectRequest
                    .builder()
                    .key(keyName)
                    .bucket(bucketName)
                    .build();

            ResponseBytes<GetObjectResponse> objectBytes = s3.getObjectAsBytes(objectRequest);
            byte[] data = objectBytes.asByteArray();

            // Write the data to a local file.
            File myFile = new File(path );
            OutputStream os = new FileOutputStream(myFile);
            os.write(data);
            System.out.println("Successfully obtained bytes from an S3 object");
            os.close();

        } catch (IOException ex) {
            ex.printStackTrace();
        } catch (S3Exception e) {
          System.err.println(e.awsErrorDetails().errorMessage());
           System.exit(1);
        }
    }
```
Read tags that belong to an object\.  

```
    public static void listTags(S3Client s3, String bucketName, String keyName ) {

         try {
             GetObjectTaggingRequest getTaggingRequest = GetObjectTaggingRequest
                    .builder()
                    .key(keyName)
                    .bucket(bucketName)
                    .build();

            GetObjectTaggingResponse tags = s3.getObjectTagging(getTaggingRequest);
            List<Tag> tagSet= tags.tagSet();
            for (Tag tag : tagSet) {
                 System.out.println(tag.key());
                 System.out.println(tag.value());
             }

         } catch (S3Exception e) {
            System.err.println(e.awsErrorDetails().errorMessage());
            System.exit(1);
        }
    }
```
Get a URL for an object\.  

```
    public static void getURL(S3Client s3, String bucketName, String keyName ) {

        try {
            GetUrlRequest request = GetUrlRequest.builder()
                    .bucket(bucketName)
                    .key(keyName)
                    .build();

            URL url = s3.utilities().getUrl(request);
            System.out.println("The URL for  "+keyName +" is "+ url);

        } catch (S3Exception e) {
            System.err.println(e.awsErrorDetails().errorMessage());
            System.exit(1);
        }
    }
```
Get an object by using the S3Presigner client object\.  

```
       public static void getPresignedUrl(S3Presigner presigner, String bucketName, String keyName ) {

        try {
            GetObjectRequest getObjectRequest =
                    GetObjectRequest.builder()
                            .bucket(bucketName)
                            .key(keyName)
                            .build();

            GetObjectPresignRequest getObjectPresignRequest = GetObjectPresignRequest.builder()
                            .signatureDuration(Duration.ofMinutes(60))
                            .getObjectRequest(getObjectRequest)
                             .build();

            PresignedGetObjectRequest presignedGetObjectRequest =
                    presigner.presignGetObject(getObjectPresignRequest);
            String theUrl = presignedGetObjectRequest.url().toString();
            System.out.println("Presigned URL: " + theUrl);
            HttpURLConnection connection = (HttpURLConnection) presignedGetObjectRequest.url().openConnection();
            presignedGetObjectRequest.httpRequest().headers().forEach((header, values) -> {
                values.forEach(value -> {
                    connection.addRequestProperty(header, value);
                });
            });

            // Send any request payload that the service needs (not needed when isBrowserExecutable is true)
            if (presignedGetObjectRequest.signedPayload().isPresent()) {
                connection.setDoOutput(true);
                try (InputStream signedPayload = presignedGetObjectRequest.signedPayload().get().asInputStream();
                     OutputStream httpOutputStream = connection.getOutputStream()) {
                    IoUtils.copy(signedPayload, httpOutputStream);
                }
            }

            // Download the result of executing the request
            try (InputStream content = connection.getInputStream()) {
                System.out.println("Service returned response: ");
                IoUtils.copy(content, System.out);
            }

        } catch (S3Exception | IOException e) {
            e.getStackTrace();
        }
       }
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/javav2/example_code/s3#readme)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/goto/SdkForJavaV2/s3-2006-03-01/GetObject) in *AWS SDK for Java 2\.x API Reference*\. 

------
#### [ JavaScript ]

**SDK for JavaScript V3**  
Create the client\.  

```
// Create service client module using ES6 syntax.
import { S3Client } from "@aws-sdk/client-s3";
// Set the AWS Region.
const REGION = "REGION"; //e.g. "us-east-1"
// Create an Amazon S3 service client object.
const s3Client = new S3Client({ region: REGION });
export { s3Client };
```
Download the object\.  

```
// Import required AWS SDK clients and commands for Node.js.
import { GetObjectCommand } from "@aws-sdk/client-s3";
import { s3Client } from "./libs/s3Client.js"; // Helper function that creates an Amazon S3 service client module.

export const bucketParams = {
  Bucket: "BUCKET_NAME",
  Key: "KEY",
};

export const run = async () => {
  try {
    // Create a helper function to convert a ReadableStream to a string.
    const streamToString = (stream) =>
      new Promise((resolve, reject) => {
        const chunks = [];
        stream.on("data", (chunk) => chunks.push(chunk));
        stream.on("error", reject);
        stream.on("end", () => resolve(Buffer.concat(chunks).toString("utf8")));
      });

    // Get the object} from the Amazon S3 bucket. It is returned as a ReadableStream.
    const data = await s3Client.send(new GetObjectCommand(bucketParams));
      return data; // For unit tests.
    // Convert the ReadableStream to a string.
    const bodyContents = await streamToString(data.Body);
    console.log(bodyContents);
      return bodyContents;
  } catch (err) {
    console.log("Error", err);
  }
};
run();
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/javascriptv3/example_code/s3#code-examples)\. 
+  For more information, see [AWS SDK for JavaScript Developer Guide](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/s3-example-creating-buckets.html#s3-example-creating-buckets-get-object)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/classes/getobjectcommand.html) in *AWS SDK for JavaScript API Reference*\. 

------
#### [ Kotlin ]

**SDK for Kotlin**  
This is prerelease documentation for a feature in preview release\. It is subject to change\.
  

```
suspend fun getObjectBytes(bucketName: String, keyName: String, path: String) {

    val request = GetObjectRequest {
        key = keyName
        bucket = bucketName
    }

    S3Client { region = "us-east-1" }.use { s3 ->
        s3.getObject(request) { resp ->
            val myFile = File(path)
            resp.body?.writeToFile(myFile)
            println("Successfully read $keyName from $bucketName")
        }
    }
}
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/kotlin/services/s3#code-examples)\. 
+  For API details, see [GetObject](https://github.com/awslabs/aws-sdk-kotlin#generating-api-documentation) in *AWS SDK for Kotlin API reference*\. 

------
#### [ PHP ]

**SDK for PHP**  
Get an object\.  

```
$s3client = new Aws\S3\S3Client(['region' => 'us-west-2', 'version' => 'latest']);

try {
    $file = $s3client->getObject([
        'Bucket' => $bucket_name,
        'Key' => $file_name,
    ]);
    $body = $file->get('Body');
    $body->rewind();
    echo "Downloaded the file and it begins with: {$body->read(26)}.\n";
} catch (Exception $exception) {
    echo "Failed to download $file_name from $bucket_name with error: " . $exception->getMessage();
    exit("Please fix error with file downloading before continuing.");
}
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/php/example_code/s3/s3_basics#code-examples)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/goto/SdkForPHPV3/s3-2006-03-01/GetObject) in *AWS SDK for PHP API Reference*\. 

------
#### [ Python ]

**SDK for Python \(Boto3\)**  
  

```
class ObjectWrapper:
    def __init__(self, s3_object):
        self.object = s3_object
        self.key = self.object.key

    def get(self):
        """
        Gets the object.

        :return: The object data in bytes.
        """
        try:
            body = self.object.get()['Body'].read()
            logger.info(
                "Got object '%s' from bucket '%s'.",
                self.object.key, self.object.bucket_name)
        except ClientError:
            logger.exception(
                "Couldn't get object '%s' from bucket '%s'.",
                self.object.key, self.object.bucket_name)
            raise
        else:
            return body
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/python/example_code/s3/s3_basics#code-examples)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/goto/boto3/s3-2006-03-01/GetObject) in *AWS SDK for Python \(Boto3\) API Reference*\. 

------
#### [ Ruby ]

**SDK for Ruby**  
Get an object\.  

```
require "aws-sdk-s3"

# Wraps Amazon S3 object actions.
class ObjectGetWrapper
  attr_reader :object

  # @param object [Aws::S3::Object] An existing Amazon S3 object.
  def initialize(object)
    @object = object
  end

  # Gets the object directly to a file.
  #
  # @param target_path [String] The path to the file where the object is downloaded.
  # @return [Aws::S3::Types::GetObjectOutput, nil] The retrieved object data if successful; otherwise nil.
  def get_object(target_path)
    @object.get(response_target: target_path)
  rescue Aws::Errors::ServiceError => e
    puts "Couldn't get object #{@object.key}. Here's why: #{e.message}"
  end
end

# Replace bucket name and object key with an existing bucket and object that you own.
def run_demo
  bucket_name = "doc-example-bucket"
  object_key = "my-object.txt"
  target_path = "my-object-as-file.txt"

  wrapper = ObjectGetWrapper.new(Aws::S3::Object.new(bucket_name, object_key))
  obj_data = wrapper.get_object(target_path)
  return unless obj_data

  puts "Object #{object_key} (#{obj_data.content_length} bytes} downloaded to #{target_path}."
end

run_demo if $PROGRAM_NAME == __FILE__
```
Get an object and report its server\-side encryption state\.  

```
require "aws-sdk-s3"

# Wraps Amazon S3 object actions.
class ObjectGetEncryptionWrapper
  attr_reader :object

  # @param object [Aws::S3::Object] An existing Amazon S3 object.
  def initialize(object)
    @object = object
  end

  # Gets the object into memory.
  #
  # @return [Aws::S3::Types::GetObjectOutput, nil] The retrieved object data if successful; otherwise nil.
  def get_object
    @object.get
  rescue Aws::Errors::ServiceError => e
    puts "Couldn't get object #{@object.key}. Here's why: #{e.message}"
  end
end

# Replace bucket name and object key with an existing bucket and object that you own.
def run_demo
  bucket_name = "doc-example-bucket"
  object_key = "my-object.txt"

  wrapper = ObjectGetEncryptionWrapper.new(Aws::S3::Object.new(bucket_name, object_key))
  obj_data = wrapper.get_object
  return unless obj_data

  encryption = obj_data.server_side_encryption.nil? ? "no" : obj_data.server_side_encryption
  puts "Object #{object_key} uses #{encryption} encryption."
end

run_demo if $PROGRAM_NAME == __FILE__
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/ruby/example_code/s3#code-examples)\. 
+  For API details, see [GetObject](https://docs.aws.amazon.com/goto/SdkForRubyV3/s3-2006-03-01/GetObject) in *AWS SDK for Ruby API Reference*\. 

------
#### [ Rust ]

**SDK for Rust**  
This documentation is for an SDK in preview release\. The SDK is subject to change and should not be used in production\.
  

```
pub async fn download_object(client: &Client, bucket_name: &str, key: &str) -> Result<(), Error> {
    let resp = client
        .get_object()
        .bucket(bucket_name)
        .key(key)
        .send()
        .await?;
    let data = resp.body.collect().await;
    println!(
        "Data from downloaded object: {:?}",
        data.unwrap().into_bytes().slice(0..20)
    );

    Ok(())
}
```
+  Find instructions and more code on [GitHub](https://github.com/awsdocs/aws-doc-sdk-examples/tree/main/rust_dev_preview/s3#code-examples)\. 
+  For API details, see [GetObject](https://docs.rs/releases/search?query=aws-sdk) in *AWS SDK for Rust API reference*\. 

------

For a complete list of AWS SDK developer guides and code examples, see [Using this service with an AWS SDK](UsingAWSSDK.md#sdk-general-information-section)\. This topic also includes information about getting started and details about previous SDK versions\.