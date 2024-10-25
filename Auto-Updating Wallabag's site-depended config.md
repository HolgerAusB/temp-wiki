## Auto-Updating Wallabag's site-depended config

Wallabag only updates its config files as part of releases for the wallabag package. However, since these config files originate from another project, they can also be loaded from the original repo into a separate folder and automatically kept up to date there.

#### Prerequisites:

- A self-hosted instance of wallabag    
- GIT package installed - or - a self-hosted Fulltext-RSS instance (current versions need a paid license)

#### Do NOT try to...

...auto-update config files within wallabag's path under `vendor/j0ker/graby-site-config/` this would be overwritten when installing a new wallabag update.

## How-to:

Go to a folder to which the wallabag user, e.g www-data, has read permission. The second command automatically creates a subfolder and retrieves the files. The folder should be outside the path of wallabag itself, it also should not exposed to the public internet by apache2, nginx, etc:
```bash
cd /var/www/
sudo git clone https://github.com/fivefilters/ftr-site-config
```

for later updating, you could just go to the sub-folder ftr-site-config again and

```bash
cd /var/www/ftr-site-config
sudo git pull
```

Now you need to inform wallabag about a second config-file folder. Within your wallabag installation path, edit the file `app/configs/services.yml` and navigate to this part:

```yml
Graby\SiteConfig\ConfigBuilder:
   arguments:
      $config: {}
```

now fill the `{}` of third line with the parameter `site_config: []` square brackets, because it is an array:

```yml
Graby\SiteConfig\ConfigBuilder:
   arguments:
      $config: { site_config: ['/var/www/ftr-site-config']}
```

And now you just need a cron-job to perform the update-check every day or hour:

File `/etc/cron.daily/update_ftr-configs` (no .sh suffix!)  **- or -**
File `/etc/cron.hourly/update_ftr-configs`
```bash
cd /var/www/ftr-site-config
git pull
```

### Fulltext-RSS (FTR) self-hosters...

...do not need to deal with GIT themselves. You can just link the paths from FTR, which should on the same machine, of course. FTR uses two folders. Under .../standard you find the original github repo, which we later auto-update via cron job. And under .../custom the admin can write or modify his/her own config files or test it before sending them to fivefilter's repo. And as we learned previously, the site_config parameter is an array, so we can link both to wallabag 😀 The last path in the array wins. And as I want the custom folder to be highest priority, it has to be the last in the row.

File `app/configs/services.yml`
```yml
Graby\SiteConfig\ConfigBuilder:
   arguments:
      $config: { site_config: ['/var/www/path_to/ftr/site_config/standard', '/var/www/path_to/ftr/site_config/custom'] }
```

For the cron job, first navigate with your web browser to the admin area of your Fulltext-RSS server. In the 'Update patterns' section you'll find a coded link for 'Automatic updates'. Every time you call this link, FTR is triggered to get site_config updates. Use this for your cron job.

File `/etc/cron.hourly/update_ftr-configs`

```
curl https://ftr.example.com/admin/update.php?key=1abcd1234
```

### Tip: Clear the cache of wallabag

Wallabag does not use a new config immediately after it has been updated. Because of caching, it could last about 10 minutes without any fetches before wallabag is re-reading the configs. That is no problem for the auto-updates. But if you want to write, edit or test your own configs, you won't wait 10 minutes after every change. You can clear the cache, with the following bash command. The given user 'www-data' is for Debian based systems, you may change this):

`sudo -u www-data php /path-to-wallabag/bin/console --env=prod cache:clear`

### Dockerized wallabag

Someone with docker skills, please add some helpful things here, like how to bring the git folder into the container, edit the config or how to clear wallabag's cache.