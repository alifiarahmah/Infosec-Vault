---
# yaml-language-server: $schema=schemas\page.schema.json
Object type:
    - Page
Backlinks:
    - File Inclusion
Creation date: "2025-10-25T06:09:00Z"
Created by:
    - Stable Quark
id: bafyreia6jr7pwhb566n7lj5eylujupxdy5nvo6xmgsldnfmjoxwkvl6tqy
---
# File Inclusion - Skills Assessment   

This module turns out to have a decent difficulty compared to other modules in Web Penetration Tester Path, as this is the first time I ran out of time and had to respawn the machine 😅 Okay let's get started   
When we first launch the target with Burp, it shows Sumace, an IT consultant service website.   

![f8e4aef230d8a404e93b42a9fb92a304649c3458f82e5dcda76f50fb66e5efdb](f8e4aef230d8a404e93b42a9fb92a304649c3458f82e5d.png)    

On the top right side there are navigation to Home, Contact, and Apply. Let's navigate to Apply first.   

![238e5f55f7d0f50b3779ccca876aec3de3850546769e2f78ee917a737b29e320](238e5f55f7d0f50b3779ccca876aec3de3850546769e2f.png)    

A form. Interesting. Let's try uploading random names and see what happens on back end.   

![ca57e95882c34a3e0dfc28296406678efe3bc608b820076c1af6933ebe36b9b2](ca57e95882c34a3e0dfc28296406678efe3bc608b82007.png)    
![1effca8d7ba00a61726ffe8f7a7bdc2530d1c7115344cd71b7df140a3b54db81](1effca8d7ba00a61726ffe8f7a7bdc2530d1c7115344cd.png)    
![8ed78dd0cf24c8c15222fd18865671d5d596457477b77e3958ea6dd077287962](8ed78dd0cf24c8c15222fd18865671d5d596457477b77e.png)    

It seems to do normal request, the firstname we use as input on POST `/api/application.php` seems to reflects to `thanks.php?n=firstname`. But it seems that we discovers a path, `/api/image.php?p=a4cbc9532b6364a008e2ac58347e3e3c` , which also appears in `index.php` as the company icon when we browse it.   

![7b1cfb5c26cd547b79f8728ff95da1ace914b2bf69ff6bb31d2d793a88c17268](7b1cfb5c26cd547b79f8728ff95da1ace914b2bf69ff6b.png)    
![d04c4d9f2e78662b8bf3b7bfa7146efe51e2304525bacde2a0f179861598239e](d04c4d9f2e78662b8bf3b7bfa7146efe51e2304525bacd.png)    

Hmm, it returns empty? Maybe our special symbols got filtered. Let's do a workaround by replacing `../` with `....//` as explained in module.   

![28dfcfe848176d20dc4ede3a1af619a91f1ac0a44a56ccd1964201539f2e6e05](28dfcfe848176d20dc4ede3a1af619a91f1ac0a44a56cc.png)    

Wow, it works! Now Let's try to read another files that we have identified. Let's start with the very own /api/image.php first.   

![ed139d5a605da1f2bbbc0d2912139474b1576394288e2a5105f6e648648f785e](ed139d5a605da1f2bbbc0d2912139474b1576394288e2a.png)    
![8d441eb37f9cf44443f5ef3c6c3a917eafe14d65f8c1b75af9824817f8600d16](8d441eb37f9cf44443f5ef3c6c3a917eafe14d65f8c1b7.png)    
Wow, seems obvious that there's LFI vulnerability here, as it has `file\_get\_contents` function with `$path` variable that only replace bare `../` with empty string.   
In `/api/application.php`, we can see that the application will encode the file into MD5 then put the file in `/uploads` folder.   
`index.php` and `apply.php` also don't give any useful PHP code for us. But when we try to read the seemingly-static `/contact.php` (the one we use in POST to submit form), there's this interesting PHP snippet:   
```
<?php
    $region = "AT";
    $danger = false;

    if (isset($_GET["region"])) {
        if (str_contains($_GET["region"], ".") || str_contains($_GET["region"], "/")) {
            echo "'region' parameter contains invalid character(s)";
            $danger = true;
        } else {
            $region = urldecode($_GET["region"]);
        }
    }

    if (!$danger) {
        include "./regions/" . $region . ".php";
    }
?>
```
Which reveals that there's hidden `region` parameter in `contact.php`. The code itself reveals include function, which based on the module, allows us to execute code. As the said it will URL decode the region parameter (), let's try to put `../index` that is double encoded as parameter.   
Based on the code, the browser would decode at first step, and the result is the one that will be checked by the code as safe/unsafe. If it is considered safe, it will decode once more by the code. Let's throw away the `.php` extension as it will   
appended by the code itself.   
![500403e84def123d619dadd049242e42341e059dd0be42084dbf46a897494ee6](500403e84def123d619dadd049242e42341e059dd0be42.png)    
![963d63471bb6334296c08fb9974333f6676c17fa01b19e891912b8ce501d3a5e](963d63471bb6334296c08fb9974333f6676c17fa01b19e.png)    
Amazing! It returns the index.php page. Now because it really has the LFI vulnerability, let's try to upload a webshell via the Apply page.   
![9726437f57ffc240b54ddda40153f74b80bf5abbf5715acdee0620b6c31d2b3f](9726437f57ffc240b54ddda40153f74b80bf5abbf5715a.png)    
![b662f51d072d3134beaec8413c6b87064abb9352d6cb8b21c81e0b09cc03ab0a](b662f51d072d3134beaec8413c6b87064abb9352d6cb8b.png)    
Now, based on `api/application.php` code we saw earlier, the code will be stored on uploads folder with name md5sum of `shell.php` file as it uses `[tmp\_file]` on the code. The file, not just the file name. Let's calculate it in the terminal.   
![5fc635afadff2566fc4d050bee6100382fcf44ebccc73122282c7489ca8bb166](5fc635afadff2566fc4d050bee6100382fcf44ebccc731.png)    
Considering that `region` parameter use `region/` folder as current directory. so the payload would be `../uploads/25a452927110e39a345a2511c57647f2`. Let's try it with `/api/image.php` first to test whether the file could get included   
![6476db7b3964be5cccec66c539c67a4ccf68ed2d36cbae2c9afe9846c6278a23](6476db7b3964be5cccec66c539c67a4ccf68ed2d36cbae.png)    
Yay it works!!! Now let's try to double-encode   
`uploads/3a9f80e2a01d6f5248e860cd25efa02c&cmd=id` and put in `contact.php`   
![2be16faaf6d5cede1f2bbf6f438dfc8252aaf57453845c020696a8ced3e4a51f](2be16faaf6d5cede1f2bbf6f438dfc8252aaf57453845c.png)    
![c535764013302bb55ad331832f28016f67b64a6e6cbce28501a54c27dc6b1d21](c535764013302bb55ad331832f28016f67b64a6e6cbce2.png)    
Yay!!! Now let's upload another file with command cat /flag\_09ebca.txt   
![3813964f4b512befa1698f7037f8087994c714f4e711767cde8122b3f4541780](3813964f4b512befa1698f7037f8087994c714f4e71176.png)    
Whew finally we got the flag!!!   
That's it. Thanks for reading ^^   
**Note**: After thinking through once more, it seems that I also double-encoded the `cmd` parameter used for the payload, thus the shell doesn't detect the input 😅 I'll redo it later   
