---
title: C# 计算文件 MD5
date: 2018-04-10 16:58:18
categories:
  - 400-编程
  - .Net
---

计算文件 MD5

<!-- more -->

```CSharp
static string CalculateMd5(string filename)
{
    using (var md5 = System.Security.Cryptography.MD5.Create())
    {
        using (var stream = File.OpenRead(filename))
        {
            var hash = md5.ComputeHash(stream);
            return BitConverter.ToString(hash).Replace("-", "").ToLowerInvariant();
        }
    }
}
// Verify a hash against a string.
static bool VerifyMd5Hash(string filename, string hash)
{
    // Hash the input.
    string hashOfInput = CalculateMd5(filename);

    // Create a StringComparer an compare the hashes.
    StringComparer comparer = StringComparer.OrdinalIgnoreCase;

    if (0 == comparer.Compare(hashOfInput, hash))
    {
        return true;
    }
    else
    {
        return false;
    }
}
```