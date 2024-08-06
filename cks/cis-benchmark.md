# CIS
Center for Internet Security. It provides th best practices to follow for various os, kubernetes, clouds etc. 

One way is to register to the cis and download the pdf with best practices and go to each eaqch section verify against your installation.

Another way is to run the tool prpvided by CIS. **CAS CAT Lite** or **CIS-CAT Pro**

## Benchmarks

### Ubuntu
Run with beloe command using **CIS-CAT-Pro**
````
./Assessor-CLI.sh -i -rd /var/www/html/ -nts -rp index
````
- i - interactive
- nts - no timestamp
- rp - report file name. default format is html
