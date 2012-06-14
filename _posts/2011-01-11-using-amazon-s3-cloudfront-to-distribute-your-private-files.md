---
title: "Using Amazon S3 and Cloudfront to distribute your private files"
date: 2011-01-11
layout: post
---

Simple 10 minute guide that will direct you on how to create a link on amazon cloudfront that expires after 30 seconds (or whatever time limit you set) to stream or deliver your private downloads or videos.

<h2>1. Setup account</h2>
First step is to setup an Amazon AWS account here: <a href="https://aws-portal.amazon.com/gp/aws/developer/registration/index.html">https://aws-portal.amazon.com/gp/aws/developer/registration/index.html</a>

Amazon have recently introduced a free tier to all its AWS services with very generous data allowances, you can read all about it here: <a href="http://aws.amazon.com/free/">http://aws.amazon.com/free/</a>

In addition to these services, the <a href="http://aws.amazon.com/console">AWS Management Console</a> is available at no charge to help you build and manage your application on AWS.
<h2>2. Create an S3 bucket</h2>
Now that you have created your amazon S3 account, login to the Management Console and create a new bucket. NOTE: the name needs to be unique...  <strong>AND PLEASE AVOID USING FULLSTOPS</strong>. If you use fullstops, you will start getting “The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel. The remote certificate is invalid according to the validation procedure” when accessing the buckets using the AWS library.  This has been an issue for years and still has not been fixed, something to do with the different between the EU/US storage locations. There are workarounds, but for simplicity just avoid using them.

Click the upload button in your bucket you just created and choose some files. On the window, dont click “start upload” yet, click on “set permissions”.  Click “Add More Permissions” and grant “Authorised Users”  access to download the file by checking the “Open/Download” checkbox. Start the upload.

I personally named my filenames as "SoftwareName_Version.exe" because the AWS library allows simple queries on request to get all items that begin with a certain phrase etc, but whatever naming scheme works for you.

<h2>3. Create a new key pair</h2>
Next step, <a href="https://aws-portal.amazon.com/gp/aws/developer/account/index.html?ie=UTF8&amp;action=access-key#keypair_block">Click Here</a> and Create a new cloudfront KeyPair, <strong>and make sure you save the .pem file that you are returned when you create it</strong>. There is no way to re-download this file, so if you miss saving that file, remove the key and start again. Save this file somewhere safe as you will need it soon to sign the URL with.

<strong>Also take note of your “Key Pair ID” as you will need this to sign the URL also.</strong>

<h2>4. Setup a Cloudfront distribution of files in the bucket</h2>
Here is the annoying part: Amazon hasn’t implemented the feature to allow you to create a private distribution from inside the AWS management console. You can do it through web service calls... but the much much easier option is to <a href="http://www.bucketexplorer.com/be-download.html">download bucket explorer</a> (free 30 day trial... and you only need to setup the distribution once so it should be more than enough time.)

Follow this guide to setup your own private distribution, as they explain it better than I ever could:
<a href="http://www.bucketexplorer.com/documentation/cloudfront--how-to-manage-private-content-for-amazon-cloudfront-distribution.html">http://www.bucketexplorer.com/documentation/cloudfront--how-to-manage-private-content-for-amazon-cloudfront-distribution.html</a>

<h2>5. Signing the URL</h2>
I could not for the life of me work out how to create a signed URL using the AWS library provided by amazon... but I did come across this very useful class that can do it all for me real easily. I can’t remember where it came from but I came across it on the internet somewhere a few months ago. If anyone knows where it’s from, please let me know and I will credit them with the initial code. It also uses the openSSL library. I have hacked it up a bit and added support for relative path load in of the PEM file.

<pre class="prettyprint">
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Security.Cryptography;
using System.IO;

namespace PMI.AWS.CloudFront
{
    public class SignedURL
    {
        /// &lt;summary&gt;
        /// This Generates a signed http url using a canned policy.
        /// To create the PEM file and KeyPairID please visit https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key
        /// &lt;/summary&gt;
        /// &lt;param name="resourceURL"&gt;The URL of the distribution item you are signing.&lt;/param&gt;
        /// &lt;param name="expiryTime"&gt;UTC time to expire the signed URL&lt;/param&gt;
        /// &lt;param name="pemFileLocation"&gt;The path and name to the PEM file. Can be either Relative or Absolute.&lt;/param&gt;
        /// &lt;param name="keypairId"&gt;The ID of the private key used to sign the request&lt;/param&gt;
        /// &lt;param name="urlEncode"&gt;Whether to URL encode the result&lt;/param&gt;
        /// &lt;returns&gt;A String that is the signed http request.&lt;/returns&gt;
        public static string GetPreSignedURLWithPEMFile(string resourceURL, DateTime expiryTime, string pemFileLocation, string keypairId, bool urlEncode)
        {
            if (pemFileLocation.StartsWith("~"))
            {
                var baseDirectory = System.AppDomain.CurrentDomain.BaseDirectory;
                pemFileLocation = Path.GetFullPath(baseDirectory + pemFileLocation.Replace("~", string.Empty));
            }

            System.IO.StreamReader myStreamReader = new System.IO.StreamReader(pemFileLocation);
            string pemKey = myStreamReader.ReadToEnd();
            pemKey = pemKey.Replace("\n", "");
            return GetPreSignedURLWithPEMKey(resourceURL, expiryTime, pemKey, keypairId, urlEncode);
        }

        /// &lt;summary&gt;
        /// This Generates a signed http url using a canned policy.
        /// To create the PEM file and KeyPairID please visit https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key
        /// &lt;/summary&gt;
        /// &lt;param name="resourceURL"&gt;The URL of the distribution item you are signing.&lt;/param&gt;
        /// &lt;param name="expiryTime"&gt;UTC time to expire the signed URL&lt;/param&gt;
        /// &lt;param name="pemFileLocation"&gt;The actual pem file&lt;/param&gt;
        /// &lt;param name="keypairId"&gt;The ID of the private key used to sign the request&lt;/param&gt;
        /// &lt;param name="urlEncode"&gt;Whether to URL encode the result&lt;/param&gt;
        /// &lt;returns&gt;A String that is the signed http request.&lt;/returns&gt;
        public static string GetPreSignedURLWithPEMKey(string resourceURL, DateTime expiryTime, string keyPEM, string keypairId, bool urlEncode)
        {
            string xmlKey = JavaScience.opensslkey.DecodePEMKey(keyPEM);
            return GetPreSignedURLWithXMLKey(resourceURL, expiryTime, xmlKey, keypairId, urlEncode);
        }

        /// &lt;summary&gt;
        /// This Generates a signed http url using a canned policy.
        /// To create the PEM file and KeyPairID please visit https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key
        /// &lt;/summary&gt;
        /// &lt;param name="resourceURL"&gt;The URL of the distribution item you are signing.&lt;/param&gt;
        /// &lt;param name="expiryTime"&gt;UTC time to expire the signed URL&lt;/param&gt;
        /// &lt;param name="pemFileLocation"&gt;The actual pem file&lt;/param&gt;
        /// &lt;param name="keypairId"&gt;The ID of the private key used to sign the request&lt;/param&gt;
        /// &lt;param name="urlEncode"&gt;Whether to URL encode the result&lt;/param&gt;
        /// &lt;returns&gt;A String that is the signed http request.&lt;/returns&gt;
        public static string GetPreSignedURLWithXMLKey(string resourceURL, DateTime expiryTime, string keyXML, string keypairId, bool urlEncode)
        {
            long expiry = (long)(expiryTime.ToUniversalTime() - new DateTime(1970, 1, 1)).TotalSeconds;
            string policy = String.Format(
                @"&#123;&#123;""Statement"":[&#123;&#123;""Resource"":""&#123;0&#125;"",""Condition"":&#123;&#123;""DateLessThan"":&#123;&#123;""AWS:EpochTime"":&#123;1&#125;&#125;&#125;&#125;&#125;&#125;&#125;]&#125;&#125;",
                resourceURL, expiry);

            RSACryptoServiceProvider rsa = new RSACryptoServiceProvider();
            RSACryptoServiceProvider.UseMachineKeyStore = false;
            rsa.FromXmlString(keyXML);

            string signature = UrlSafe(rsa.SignData(Encoding.UTF8.GetBytes(policy), new SHA1CryptoServiceProvider()));

            string formatStr = urlEncode ?
            "{0}%3fExpires%3d{1}%26Signature%3d{2}%26Key-Pair-Id%3d{3}" :
            "{0}?Expires={1}&Signature={2}&Key-Pair-Id={3}";

            return string.Format(formatStr, resourceURL, expiry, signature, keypairId);
        }

        private static string UrlSafe(byte[] data)
        {
            return Convert.ToBase64String(data)
            .Replace('+', '-').Replace('=', '_').Replace('/', '~');
        }
    }
}
</pre>

and you also need the openSSL library from here: <a href="http://www.jensign.com/opensslkey/opensslkey.cs" target="_blank">http://www.jensign.com/opensslkey/opensslkey.cs</a>

<h2>6. Add your PEM file to your project</h2>
Create a new folder, just create a new guid and name your folder that. For eg, “9467a84d-39f8-464e-a284-8535bd0f8c44”  or whatever you want. Just make sure its unique and cant be guessed!!!

In that file, place your PEM file that you downloaded in step 3.

of course if your site is hosted on VPS and you have access to the local filesystem, store your PEM file there and use that path (yep, the helper class below is smart enough to pick up whether its a relative or absolute path)

<h2>7. Creating the Signed url</h2>
<pre class="prettyprint">

DateTime expires = DateTime.UtcNow.AddSeconds(30);
string signedURL = PMI.AWS.CloudFront.SignedURL.GetPreSignedURLWithPEMFile("[Cloudfront Distribution URL]/" + [Filename Of Item In Bucket], expires, "~/[GUID Folder Created in Step 6]/[PEM Key Filename inside folder]", "[Key Pair ID]", false);

</pre>

And thats it! "signedURL" is the URL of the file that will expire in 30 seconds and return an "unauthorized" message.

All you need to do now to add more items to your distribution is add items to your bucket and make sure the files have “Authenticated users” checked to view and download. this can be used to secure streaming videos, only allow users to have access to a file download for 2 hours etc etc. plus with cloudfronts speed... you know that whoever is looking at your content is going to be getting it the fastest way possible.