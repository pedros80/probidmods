## PHP Probid - Backup Database From Cron
[Peter Somerville 2013](http://www.pedros-stuffs.com) - peterwsomerville@gmail.com

### License
GNU GPL - See LICENSE.txt for more information


### About
This mod allows the administrator to activate a daily backup of their main database.

The backup can be optionally gzipped to save space and is stored in a protected directory on the server

### Installation

First create the directory `/cron_jobs/db_dumps/` and give it permissions to allow apache to create files.

### MySQL Operations


```sql 
	ALTER TABLE `pitmart_gen_setts` ADD `enable_db_backup` TINYINT( 1 ) NOT NULL DEFAULT '0';
  ALTER TABLE `pitmart_gen_setts` ADD `gzip_db_backup` TINYINT( 1 ) NOT NULL DEFAULT '0';
```


### File Operations

### `/cron_jobs/main_cron_daily.php`

**After `include`s at top of file**

**Add**

```php
    if ($setts['enable_db_backup']) {
       require 'dbdump.php';

       $dbBU = new dbBackup($db_host, $db_username, $db_password, $db_name);
       $dbBU->setts = $setts;
       $dbBU->databaseDump();
    }
```


### `/ADMIN_DIRECTORY/templates/index.tpl.php`


**Find**

```html
    <img src="images/a.gif" align="absmiddle" vspace="2"> <a href="word_filter.php"><?php echo AMSG_WORD_FILTER;?></a> <br>
```

**Above Add**

```html
    <img src="images/a.gif" align="absmiddle" vspace="2"> <a href="db_backup.php">Cron DB Backup</a> <br>
```

### `/ADMIN_DIRECTORY/templates/leftmenu.tpl.php`

**Find**

```html
	<div class="fsidew">
```

**Below Add**

```html
	<div class="alink"><a href="db_backup.php">Cron DB Backup</a></div>
```

**Find**

```php
    stristr($_SERVER['PHP_SELF'], "word_filter.php")||
```

**Above Add**

```php
    stristr($_SERVER['PHP_SELF'], "db_backup.php")||
```

## New Files

### `/cron_jobs/dbdump.php`


```php

<?php

    /**
     * dbBackup used by main_cron_daily to write a database backup
     * to site/cron_jobs/db_dumps. If gzip compression is available,
     * use it on backup file.
     */

    class dbBackup {

       var $setts;
       var $host;
       var $user;
       var $pass;
       var $db_name;
       var $path;

       public function __construct($host, $user, $pass, $db_name) {
          $this->host = $host;
          $this->user = $user;
          $this->pass = $pass;
          $this->db_name = $db_name;
          $this->path = getcwd().'/db_dumps/';
       }

       public function databaseDump() {
          $this->_doDump($this->db_name);
       }

       private function _doDump($name) {
          $theTime = date('D');
          $theFile = $this->path.$name.$theTime.'.sql';

          system("mysqldump -h $this->host -u $this->user -p$this->pass $this->db_name > $theFile");

          if (extension_loaded('zlib') && $this->setts['gzip_db_backup']) {
             $gzfile = $this->path.$this->db_name.$theTime.'.gz';
             $fp = gzopen($gzfile, 'w9');
             gzwrite($fp, file_get_contents($theFile));
             gzclose($fp);
             unlink($theFile);
          }
       }

    }
    ?>
```


### `/ADMIN_DIRECTORY/db_backup.php`

```php
<?php
    session_start();

    define ('IN_ADMIN', 1);
    define ('IN_SITE', 1);

    include_once ('../includes/global.php');

    if ($session->value('adminarea') != 'Active') {
       header_redirect('login.php');
    }
    else
    {
       include_once ('header.php');

       $msg_changes_saved = '<p align="center" class="contentfont">'.AMSG_CHANGES_SAVED.'</p>';

       $template->set('zlib_enabled', extension_loaded('zlib'));
       $template->set('file_name', $db_name.date('D'));

       if (isset($_POST['form_save_settings'])) {
          $template->set('msg_changes_saved', $msg_changes_saved);

          $save_vars = $db->rem_special_chars_array($_POST);

          $sql_save_settings = $db->query("UPDATE ".DB_PREFIX."gen_setts
                                           SET
                                           enable_db_backup='".intval($save_vars['enable_db_backup'])."',
                                           gzip_db_backup='".intval($save_vars['gzip_db_backup'])."'");
       }
       $setts_tmp = $db->get_sql_row("SELECT * FROM ".DB_PREFIX."gen_setts");

       $template->set('setts_tmp', $setts_tmp);

       $header_section = 'CRON DB BACKUP SETTINGS';
       $subpage_title = 'CRON DB BACKUP SETTINGS';

       $template->set('header_section', $header_section);
       $template->set('subpage_title', $subpage_title);

       $template_output .= $template->process('db_backup.tpl.php');

       include_once ('footer.php');

       echo $template_output;
    }
    ?>
```


### `/ADMIN_DIRECTORY/templates/db_backup.tpl.php`

```html
 <?php

    if ( !defined('INCLUDED') ) { die("Access Denied"); }
    ?>
    <div class="mainhead"><img src="images/general.gif" align="absmiddle"> <?php echo $header_section;?></div>
    <?php echo $msg_changes_saved;?>
    <table width="100%" border="0" cellpadding="0" cellspacing="0">
      <tr>
         <td width="4"><img src="images/c1.gif" width="4" height="4"></td>
         <td width="100%" class="ftop"><img src="images/pixel.gif" width="1" height="1"></td>
         <td width="4"><img src="images/c2.gif" width="4" height="4"></td>
       </tr>
    </table>
    <form name="form_db_backup" method="post" action="db_backup.php" enctype="multipart/form-data">
    <table width="100%" border="0" cellpadding="3" cellspacing="3" class="fside">
       <tr class="c3">
          <td colspan="2"><img src="images/subt.gif" align="absmiddle" hspace="4" vspace="2"> <b><?php echo strtoupper($subpage_title);?></b></td>
       </tr>
      <tr class="c1">
          <td width="200" align="right"><b>Enable database backup?</b></td>
          <td>
             <input type="radio" name="enable_db_backup" id="enable_db_backup_yes" value="1" <?php echo ($setts_tmp['enable_db_backup'] == 1) ? 'checked="checked"' : '' ;?>>
             <label for="enable_db_backup_yes"><?php echo GMSG_YES;?></label>
             <input type="radio" name="enable_db_backup" id="enable_db_backup_no" value="0" <?php echo ($setts_tmp['enable_db_backup'] == 0) ? 'checked="checked"' : '' ;?>>
             <label for="enable_db_backup_no"><?php echo GMSG_NO;?></label>
          </td>
       </tr>
       <tr>
          <td class="explain" align="right"><img src="images/info.gif"></td>
          <td class="explain">If enabled the database will be backed up by the main_cron_daily</td>
       </tr>
       <tr class="c1">
          <td align="right"><b>gzip file?</b></td>
          <td>
             <input type="radio" name="gzip_db_backup" id="gzip_db_backup_yes" value="1" <?php echo ($setts_tmp['gzip_db_backup'] == 1) ? 'checked="checked"' : '' ;?> <?php echo (!$zlib_enabled) ? 'disabled="disabled" readonly="readonly"' : '';?>>
             <label for="gzip_db_backup_yes"><?php echo GMSG_YES;?></label>
             <input type="radio" name="gzip_db_backup" id="gzip_db_backup_no" value="0" <?php echo ($setts_tmp['gzip_db_backup'] == 0) ? 'checked="checked"' : '' ;?> <?php echo (!$zlib_enabled) ? 'disabled="disabled" readonly="readonly"' : '';?>>
             <label for="gzip_db_backup_no"><?php echo GMSG_NO;?></label>
          </td>
       </tr>
       <tr>
          <td class="explain" align="right"><img src="images/info.gif"></td>
          <td class="explain">
             If the zlib module is installed and enabled on the server the file can be optionally gzipped so it will take up less space on the server.
             <br>If this option is selected and zlib is enabled the <b><?php echo $file_name;?>.sql</b> file will be replaced by the gzipped version.
             <br>zlib module is currently <b><?php echo ($zlib_enabled) ? 'enabled' : 'disabled';?></b> on this server.
          </td>
       </tr>
       <tr align="center">
          <td colspan="2" valign="top"><input type="submit" name="form_save_settings" value="<?php echo AMSG_SAVE_CHANGES;?>"></td>
       </tr>
    </table>
    </form>
    <table width="100%" border="0" cellpadding="0" cellspacing="0">
       <tr>
          <td width="4"><img src="images/c3.gif" width="4" height="4"></td>
          <td width="100%" class="fbottom"><img src="images/pixel.gif" width="1" height="1"></td>
          <td width="4"><img src="images/c4.gif" width="4" height="4"></td>
       </tr>
    </table>
```


### `/cron_jobs/.htaccess`

```
    deny from all
    allow from YOUR.SERVER.IP.ADDRESS
    allow from 127.0.0.1
```