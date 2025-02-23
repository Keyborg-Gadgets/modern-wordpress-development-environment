# A modern word press environment with livereload that actually works
TLDR (but you probably will want to modify the gulpfile at some point):
```
git clone https://github.com/Keyborg-Gadgets/modern-wordpress-development-environment.git
cd modern-wordpress-development-environment
export  MY_USER=$(whoami); export MY_UID=$(id -u)
mkdir -p wordpress mariadb; docker-compose up -d --build --remove-orphans --force-recreate; while ! docker logs wordpress 2>&1 | grep -o "WordPress setup finished"; do sleep 1; done; echo "\nYour site should now be ready\n" 
```

### A Quick Test
> NOTE: The new versions of wp don't ship with scss base themes. [onepress](https://wordpress.org/themes/onepress/) isn't bad. This env works. [Local](https://localwp.com/) works out of the box, but I still like my dev env more. It's so easy to hack up a single page. I use export plugins and then migrate to localwp. Then use [simply static](https://simplystatic.com/) to export to gh-pages. Like the work flow a lot. 
Open your browser to local host and then in a terminal:
```
sed -i.bak -e "s/flex-wrap: wrap;/background-color: red;/g" wordpress/wp-content/themes/twentytwentyone/assets/sass/06-components/header.scss 
```

The header should turn red.


# Gulpfile

This was originally a gruntfile but apparently fswatch and the disk events in a docker container/and or mounts don't play nice.  

Gulp watch has polling which rocks your cpu, but its the only way this works. 

I Intentionally use cli sass. It seems to be the fastest out of all the implementations vs ruby sass, node sass, etc. I feel this beats out any contrib. You're talking 1.x seconds vs sub .5.

I think this is pretty self explanatory at a glance:
```
gulp.watch('/wordpress/**/*.scss', {interval: 50, usePolling: true}, series(sass, lr));
```
It watches scss files across all directories under wordpress. You could probably save some cycles by just watching the theme asset files.It then run live reload. The advantage to using the official contrib and making properly chained tasks is that you can reload specific files. This works well for me though, I just reload index.php on changes. 

Then we have the sass portion

```
const stdout = execSync('bash -c "sass --no-source-map /wordpress/wp-content/themes/twentytwentyone/assets/sass/style.scss /wordpress/wp-content/themes/twentytwentyone/style.css"');
```

I could probably clean this up with some base paths, but different themes have different asset locations, so you're probably going to have to change this anyhow. Again though, works very well for what I need.

