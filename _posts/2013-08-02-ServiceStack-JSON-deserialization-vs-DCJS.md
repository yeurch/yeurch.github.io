---
layout: post
title: "ServiceStack JSON Deserialization vs DataContractJsonSerializer"
---

I used to have this horrid piece of code to deserialize a license stored as JSON:

    var deserializer = new DataContractJsonSerializer(typeof(License));
    using (var stream = new MemoryStream(ASCIIEncoding.Default.GetBytes(licenseJSON)))
    {
        theLicense = (License)deserializer.ReadObject(stream);
        stream.Close();
    }

Along came [ServiceStack.Text](http://www.servicestack.net/docs/text-serializers/json-csv-jsv-serializers),
and now I have this beautiful one-liner:

    theLicense = JsonSerializer.DeserializeFromString<License>(licenseJSON);

Or using an extension method ServiceStack provides results in even clearer code:

    theLicense = licenseJSON.FromJson<License>();
	
The simplicity and elegance of this just blows me away.  What were Microsoft thinking when they
came up with the DataContractJsonSerializer?