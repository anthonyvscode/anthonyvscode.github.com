---
title: "Using Amazon S3 & Cloudfront To distribute your private files: Part 2 - Shared hosting"
date: 2011-01-25
layout: post
---
So, you want to use the solution I posted <a href="http://anthonyvscode.com/2011/01/11/using-amazon-s3-cloudfront-to-distribute-your-private-files/">Here</a> in your shared hosting environment?

You're out of luck and you will start seeing this error below pop up every time you try to sign your url's (mostly... unless your with a shared hosting provider that likes to bend the rules and will change you to full trust in your IIS settings)

<strong>System.Security.SecurityException: Request for the permission of type 'System.Security.Permissions.SecurityPermission, mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089' failed.</strong>

<strong>Why does this happen?</strong>

Shared hosting providers (rightfully so) block IIS access from machine keys... and when the RSA .net library attempts to go about its business, it causes a security exception.

<strong>How do I fix this?</strong>

In my searching... there seems to be no workaround at all using the provided .net RSA library (unless you can afford a VPS and can give your IIS app full trust access... )

Except for this library, which is a compiled code replacement for the <code>RSACryptoServiceProvider</code> Service and works perfectly for what we are after: <a href="http://www.codeproject.com/KB/security/EZRSA.aspx">http://www.codeproject.com/KB/security/EZRSA.aspx</a>

I effectively got the Amazon Cloudfront URL signing working using this library, and no longer get the security exception in my live environment.

<strong>Enough rambling, gimme the code!</strong>
<h3>Step 1: convert your PEM file to an XML string.</h3>
I just setup a test page, called the<strong> GetPreSignedURLWithPEMFile </strong>function and put my break line inside the <strong>GetPreSignedURLWithPEMKey </strong>function after <strong>JavaScience.opensslkey.DecodePEMKey(keyPEM)</strong>; is called. This is your pem file converted to an xml string. Copy that xml and create a new string at the top of your SignedURL class (well really you can save it anywhere i just did it there so its 1 place to update if i need to change it).
<h3>Step 2: Download and reference the EZRSA library.</h3>
<strong>http://www.codeproject.com/KB/security/EZRSA/ezrsa_demo.zip</strong>
<h3>Step 3:Call the (edited) GetPreSignedURLWithXMLKey Function instead.</h3>
New edited GetPreSignedURLWithXMLKey function that uses the EZRSA class:

<pre class="prettyprint">

private const string xmlKey =

"&lt;RSAKeyValue&gt;" +
"    &lt;Modulus&gt;6eChle22XFSVa5LZcZrdOcGq4EKSO1mRwqccPWbtgEm2CFf8oXdkFkVO+dDryMZyYB+xACFbq0/ZD2uByLQAKw==&lt;/Modulus&gt;" +
"    &lt;Exponent&gt;AQAB&lt;/Exponent&gt;" +
"    &lt;P&gt;/l4Qiqve4fWyyWPpCn1BPlMkE3Qy02ieVuCrznQmfZM=&lt;/P&gt;" +
"    &lt;Q&gt;62DmmOSvPe9PsmEweo1R8IdB8c8d0B58f5boICyt0gk=&lt;/Q&gt;" +
"    &lt;DP&gt;1fiYn53uUlOVPrWtviYZMO1NRpQTgSTbNSevPm8URcM=&lt;/DP&gt;" +
"    &lt;DQ&gt;Ene54AkhTsS2BhLmENeBtFOIcwaDGk8qCYC3mb6nrLE=&lt;/DQ&gt;" +
"    &lt;InverseQ&gt;ZKMeZYfDw2pVD3bKKf3GMtPMMJyBm4i7pBNwg9wU2HY=&lt;/InverseQ&gt;" +
"    &lt;D&gt;jdTsKUA/lz60XshvlbWU87G/LsEwbU2kV6eAOLxyy5i/CsDw4pCUCku8SfvvEumDyVUQETGCenKrX+ocE9JUAQ==&lt;/D&gt;" +
&lt;RSAKeyValue&gt;";

public static string GetPreSignedURLWithXMLKey(string resourceURL, DateTime expiryTime, string keyXML, string keypairId, bool urlEncode)
{
    long expiry = (long)(expiryTime.ToUniversalTime() - new DateTime(1970, 1, 1)).TotalSeconds;
    string policy = String.Format(
        @"&#123;&#123;""Statement"":[&#123;&#123;""Resource"":""&#123;0&#125;"",""Condition"":&#123;&#123;""DateLessThan"":&#123;&#123;""AWS:EpochTime"":&#123;1&#125;&#125;&#125;&#125;&#125;&#125;&#125;]&#125;&#125;",
        resourceURL, expiry);

    if (string.IsNullOrEmpty(keyXML))
        keyXML = xmlKey;

    EZRSA ezrsa = new EZRSA(1024); //Amazon Cloudfront uses 1024 bit encrypted keys

    ezrsa.FromXmlString(keyXML);
    string signature = UrlSafe(ezrsa.SignData(Encoding.UTF8.GetBytes(policy), new SHA1CryptoServiceProvider()));

    string formatStr = urlEncode ?
    "{0}%3fExpires%3d{1}%26Signature%3d{2}%26Key-Pair-Id%3d{3}" :
    "{0}?Expires={1}&Signature={2}&Key-Pair-Id={3}";

    return string.Format(formatStr, resourceURL, expiry, signature, keypairId);
}
</pre>

<strong>Now all you need to do is call:</strong>

<pre class="prettyprint">
DateTime expires = DateTime.UtcNow.AddSeconds(30);
string formattedUrl = PMI.AWS.CloudFront.SignedURL.GetPreSignedURLWithXMLKey(cloudfrontUrl, expires, string.Empty, "KeyID", false);

</pre>

And you're set! the url will be signed like before, but you will have worked around the exception.

If you have anymore tips or questions feel free to comment!

Anthony 1 - Shared hosting 0