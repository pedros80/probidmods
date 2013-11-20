## PHP Probid - Custom 404/500 Error Pages
[Peter Somerville 2013](http://www.pedros-stuffs.com) - peterwsomerville@gmail.com

### License
GNU GPL - See LICENSE.txt for more information


### About
This mod will display custom server error pages for 500 internal server errors and 404 page not found errors
and email the site admin with the page which caused the error & the username and id of the user who received the error.

Assumes you are using the tinymce wysiwyg editor. This allows you to control the look and feel of your
custom contnent without the need to write any actual markup.


### Installation

### MySQL operations

```sql
   CREATE TABLE probid_server_error_text(
      404_text text,
      500_text text
   );
```


```sql
INSERT INTO probid_server_error_text (404_text, 500_text)
VALUES ('default text', 'default_text');
```


## File operations

### `/.htaccess`

**Add**

```
ErrorDocument 404 /404.php
ErrorDocument 500 /500.php
```


### `/includes/functions.php`

**Add**

```php
   function curPageURL() {
      $pageURL = 'http';
      if ($_SERVER["HTTPS"] == "on") {$pageURL .= "s";}
         $pageURL .= "://";
      if ($_SERVER["SERVER_PORT"] != "80") {
         $pageURL .= $_SERVER["SERVER_NAME"].":".$_SERVER["SERVER_PORT"].$_SERVER["REQUEST_URI"];
      } else {
         $pageURL .= $_SERVER["SERVER_NAME"].$_SERVER["REQUEST_URI"];
      }
      return $pageURL;
   }
```


### `/ADMIN_DIRECTORY/templates/index.tpl.php`

**Find**

 ```html
   <div class="mainhead"><img src="images/set.gif" align="absmiddle"><?php echo AMSG_GENERAL_SETTINGS;?></div>
      <table width="100%" border="0" cellpadding="0" cellspacing="0">
         <tr>
            <td width="4"><img src="images/c1.gif" width="4" height="4"></td>
            <td width="100%" class="ftop"><img src="images/pixel.gif" width="1" height="1"></td>
            <td width="4"><img src="images/c2.gif" width="4" height="4"></td>
         </tr>
      </table>
      <table width="100%" border="0" cellpadding="2" cellspacing="2" class="fside">
         <tr>
            <td width="100%" class="menulink">
```

**Below Add**

```html
<img src="images/a.gif" align="absmiddle" vspace="2"> <a href="general_settings.php?page=404_text">404 page text</a><br>
<img src="images/a.gif" align="absmiddle" vspace="2"> <a href="general_settings.php?page=500_text">500 page text</a><br>
```



###`/ADMIN_DIRECTORY/general_settings.php`

**Find**

```php
   switch ($_REQUEST['page'])
   {
```

**Below Add**

   ```php
   case '404_text':
      $sql_update_404 = $db->query("UPDATE ".DB_PREFIX."server_error_text SET
         404_text='".$db->add_special_chars($_POST['404_text'])."'");
      break;
   case '500_text':
      $sql_update_500 = $db->query("UPDATE ".DB_PREFIX."server_error_text SET
         500_text='".$db->add_special_chars($_POST['500_text'])."'");
      break;
```


**Find**

   ```php
   $template->set('setts_tmp', $setts_tmp);
   $template->set('layout_tmp', $layout_tmp);
   $template->set('page', $_REQUEST['page']);
   ```


**Below Add**

   
   ```php
   $template->set('text_tmp', $db->get_sql_row("SELECT ** FROM ".DB_PREFIX."server_error_text"));
   if ($_REQUEST['page'] == '404_text')
   {
      $header_section = AMSG_GENERAL_SETTINGS;
      $subpage_title = 'Text for custom 404 error page';
   }

   if ($_REQUEST['page'] == '500_text')
   {
      $header_section = AMSG_GENERAL_SETTINGS;
      $subpage_title = 'Text for custom 500 error page';
   }
   ```

-----------------------------
/ADMIN_DIRECTORY/templates/general_settings.tpl.php

Find

      <?php } else if ($page == 'strikes') { ?>

Above Add

    <?php } else if ($page == '404_text'){ ?>
       <tr class="c1">
         <td width="200" align="right">Text For 404 error page</td>
         <td><textarea id="404_text" name="404_text" class="tinymce"><?php echo $text_tmp['404_text'];?></textarea></td>
      </tr>
      <tr>
         <td align="right" class="explain"><img src="images/info.gif"></td>
         <td class="explain">The text and markup which will be displayed on a 404 not found error</td>
      </tr>
      <?php } else if ($page == '500_text'){?>
       <tr class="c1">
         <td width="200" align="right">Text For 500 error page</td>
         <td><textarea id="500_text" name="500_text" class="tinymce"><?php echo $text_tmp['500_text'];?></textarea></td>
      </tr>
      <tr>
         <td align="right" class="explain"><img src="images/info.gif"></td>
         <td class="explain">The text and markup which will be displayed on a 500 server error</td>
      </tr>


-----------------------------
/ADMIN_DIRECTORY/templates/leftmenu.tpl.php

Find

<td class="atitle" width="170"><?php echo AMSG_GEN_SETTS;?></td><td align="center" class="sh" width="50"><a title="show/hide" class="hidelayer" id="exp1_link" href="javascript: void(0);" onclick="toggle(this, 'exp1')">hide</a></td>
   </tr>
   <tr>
      <td colspan="2">
         <div id="exp1" style="padding: 5px;">
            <div><img src="images/subtop.gif" width="208" height="4"></div>
            <div class="fsidew">

Below Add

                <div class="alink"><a href="general_settings.php?page=404_text">404 page text</a></div>
                <div class="alink"><a href="general_settings.php?page=500_text">500 page text</a></div>


-----------------------------

## New Files

### add to root ADMIN_DIRECTORY

/404.php

<?php
#################################################################
## Custom page for 404 error                                   ##
#################################################################

session_start();
define ('IN_SITE', 1);

include_once ('includes/global.php');
include_once ('includes/functions.php');
include_once ('global_header.php');

$template_output .= $db->get_sql_field("SELECT 404_text FROM ".DB_PREFIX."server_error_text", "404_text");

$error_email = $db->get_sql_field("SELECT admin_email FROM ".DB_PREFIX."gen_setts", 'admin_email');

$email_body = "A 404 error has been detected...\n";
$email_body .= "\n".'page - ' . curPageURL();
$email_body .= "\n".'time - ' . show_date(CURRENT_TIME);

if ($session->value('user_id')>0){
    $email_body .= "\n".'user_id - ' . intval($session->value('user_id'));
    $email_body .= "\n".'username - ' . $session->value('username');
}
mail($error_email, '404', $email_body);

include_once ('global_footer.php');

echo $template_output;

?>


/500.php

<?php
#################################################################
## Custom page for 500 error                                   ##
#################################################################

session_start();
define ('IN_SITE', 1);

include_once ('includes/global.php');
include_once ('includes/functions.php');
include_once ('global_header.php');

$template_output .= $db->get_sql_field("SELECT 500_text FROM ".DB_PREFIX."server_error_text", "500_text");
$error_email = $db->get_sql_field("SELECT admin_email FROM ".DB_PREFIX."gen_setts", 'admin_email');

$email_body = "A 500 error has been detected...\n";
$email_body .= "\n".'page - ' . curPageURL();
$email_body .= "\n".'time - ' . show_date(CURRENT_TIME);

if ($session->value('user_id')>0){
    $email_body .= "\n".'user_id - ' . intval($session->value('user_id'));
    $email_body .= "\n".'username - ' . $session->value('username');
}
mail($error_email, '500', $email_body);

include_once ('global_footer.php');

echo $template_output;

?>

