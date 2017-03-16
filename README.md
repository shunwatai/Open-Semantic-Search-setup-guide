# Open Semantic Search
## Installation on Ubuntu 16.04
1. download the installation file

        # wget https://www.opensemanticsearch.org/download/open-semantic-search-server-ubuntu_xenial_17.03.14.deb
        
2. installation

        # sudo apt install ./open-semantic-search-server-ubuntu-*.deb
        
3. After installation stops continue by type in a q to end showing a status message of Solr installation script

## Perform full system upgrade
Since ```Search by list``` & ```Manage structure``` appeared broken on web interface. Fortunately, a full system upgrade can solve this issue.

    # sudo apt-get update
    # sudo apt dist-upgrade
    # reboot

## pre-configuration
edit line30 in the following file:
```/var/lib/opensemanticsearch/opensemanticsearch/settings.py```

    ALLOWED_HOSTS = [u'localhost',u'<serverIP>']

## Mounting target network shared folder for indexing (assume no password protected)
### Install samba packages
    # apt-get install libsmbclient-dev smbclient cifs-utils
### Mount the network drive
1. Mount it temporarily by command

        # mkdir /media/sharing/
        # mount -tcifs //<ServerIP>/sharing/ /media/sharing/ -o username=guest password=
        # ls -l /media/sharing/

2. If success, make it permanently by add a line in ```/etc/fstab```

        //<ServerIP>/sharing/ /media/sharing/ cifs username=guest,password=,uid=1000,iocharset=utf8 0 0
    unmount the network drive folder ```/media/sharing```
    
        # umount /media/sharing
    and then use following command verify
    
        # mount -a 

## OCR
### Install the packages
    # apt-get install -y tesseract-ocr-chi-tra tesseract-ocr-chi-sim tesseract-ocr-eng imagemagick 
### Enable it
edit the file ```/etc/opensemanticsearch/etl```, make sure the following 3 lines existed.

    config['plugins'].append('enhance_pdf_ocr')
    config['plugins'].append('enhance_ocr_descew')
    config['ocr_lang'] = 'eng+chi_tra+chi_sim'
    
## Web interface setting
### Set path mapping
1. Add symbolic link for share folder ```/media/sharing``` to web server working directory

        # ln -s /media/sharing /var/www/html/
2. Edit config file ```/etc/opensemanticsearch/connector-files```

    comment line23
    
        #config['mappings'] = { "/": "file:///" }
        
    uncomment line29 and edit like follow, it is a Python dict, I added 2 pairs as example:
    
        config['mappings'] = { "/media/": "http://<ServerIP or hostname>/media/","/home/user/documents/": "http://<ServerIP or hostname>/documents/" }

## Indexing
I recommend disable the OCR for first index because it would take very long time, so comment out those 3 lines in ```/etc/opensemanticsearch/etl``` which added in previous step. After the first indexing, then uncomment them and do a force reindexing

1. Index by command:

        # opensemanticsearch-index-dir /media/sharing/
        
2. Do force reindexing after enable OCR:

        # opensemanticsearch-index-dir --force /media/sharing/

## Filemonitoring
I cannot start it in daemon mode, I dont know why

    # opensemanticsearch-filemonitoring /media/sharing/victor/jobsheet/

Also, it seems using the ```inotify``` for monitoring, it only fully functional for add new files in Linux machine locally. If user add the new files to the network drive in Windows, nothing will happen, to solve it, you might set a ```cronjob``` for ```touch``` the shared folder periodically, the command likes follow:

    # touch -a /media/sharing/
    
However, I think it is just similar to set the ```cronjob``` for run the force reindexing periodically.

## Fix the web interface display results problem
By default, there are only 10 results would be showed up per page. If you want to show 100 per page, edit line 655 in the file ```/usr/share/solr-php-ui/index.php```

    if ($view=='list') {
            $limit = 100;
    }

