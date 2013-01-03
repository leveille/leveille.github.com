---
layout: post
title: "Avectra Member Management API"
date: 2008-04-18 20:25
comments: false
categories: PHP
---

Recently I worked on a project where all member data was being managed through a third party member management service, provided by a company named [Avectra](http://www.avectra.com/eweb/StartPage.aspx?site=Avectra07). The product does more than manage member data from the looks of things, but all I cared about was interaction with the member management API.

<!--more-->

I had to authenticate against an external data source, as well as query the data source for member information when needed. Avectra does have a web service API which it has named xWeb. Here is a quote from [their Wiki](http://wiki.avectra.com/index.php?title=NetFORUM):

{% blockquote Avectra http://wiki.avectra.com/index.php?title=NetFORUM Avectra Wiki %}
xWeb is netFORUM’s XML web services application. netFORUM is an enterprise level Association Management System (AMS) developed by Avectra that allows associations to manage their customers and related activities. netFORUM is used by association staff, members, and the public at large.
{% endblockquote %}

##Interacting with the API

After a successful authentication with Avectra, the authenticated user is presented with a token that they pass along with API requests. Pretty standard stuff. The Wiki does an ok job of explaining authentication and login (two different things), therefore I’ll jump right to the good stuff. If you’re stuck on authentication/weblogin please let me know.

For interacting with the API I created a class file to abstract, at least to a small degree, some of the details of consuming the service. Here it is:

``` php
<?php
	/*
	 * @author: Jason Leveille
	 * Provides access to avectra web services
	 */

	class Avectra
	{

	   //Avectra organization api username and password (not member username/password) to access web services
	   private $user = 'user';
	   private $pass = 'password';

	   //Avectra paths to xweb and eweb.  This is handy so that you can easily switch back and fourth between dev and production
	   private $xweb = 'www.mencnet.org/xweb/';
	   private $eweb = 'https://www.mencnet.org/netforummenctest/eweb/';

	   /**
	   * Grabs data associated with a member based on customer key / token
	   * @param $cst_key Customer Key to search on
	   */
	   public function getMemberDataByKey($cst_key) {

	      if(empty($cst_key))
	      {
	         return false;
	      }

	      $memberData = sprintf('https://%s:%s@%snetFORUMXML.asmx/GetIndividualInformation?IndividualKey=%s', $this->user, $this->pass, $this->xweb, $cst_key);

	      //The simplest solution is to call simplexml_load_file directly, however this was causing issues on Solaris 10, php 5.2.  Didn't have time to investigate, so I went with load_string
	      //return simplexml_load_file($memberData);

	      $data = simplexml_load_string(file_get_contents($memberData));

	      //make sure that the returned xml file actually contains data
	      if(empty($data->IndividualObject[0]->eml_address))
	      {
	         return false;
	      }

	      return $data;

	   }

	   /**
	   * Grabs data associated with a member based on member email
	   * @param $email Member email to search on
	   */
	   public function getMemberDataByEmail($email) {
	      if(empty($email))
	      {
	         return false;
	      }

	      $memberData = sprintf('https://%s:%s@%snetFORUMXML.asmx/GetQuery?szObjectName=%s&szColumnList=%s&szWhereClause=%s&szOrderBy=%s',
	         $this->user,
	         $this->pass,
	         $this->xweb,
	         'Individual',
	         'ind_first_name, ind_last_name, cst_org_name_dn, adr_city, adr_state, eml_address',
	         sprintf("eml_address = '%s'", $email),
	         "ind_last_name"
	      );

	      return simplexml_load_file($memberData);
	   }

	   /**
	   * Grabs the url of the avectra eweb application
	   */
	   public function getEwebUrl() {
	      return $this->eweb;
	   }
	}
	?>
```

So, for example, to retrieve data about a specific member, and to store that data in session variables for later use, your code might look similar to the following:

``` php
<?php
	function login_success($cst_key = null) {

	   if(!$cst_key)
	   {
	      //set error message and redirect
	   }

	   //ensure clean referer data
	   $original_referer = $this->clean->stripScripts($this->params['url']['ref']);

	   //Avectra object
	   //include avectra class file, for example, in CakePHP
	   App::import('Vendor', 'Avectra');
	   $avectra = new Avectra();

	   //ensure a clean customer key
	   $cst_key = $this->clean->stripScripts($cst_key);

	   //if we successfully load the associated user data
	   if($data = $avectra->getMemberDataByKey($cst_key))
	   {
	      $data = $data->IndividualObject[0];

	      $this->Session->write("Member.memberId", $this->clean->stripScripts($data->eml_address));
	      $this->Session->write("Member.firstName", $this->clean->stripScripts($data->ind_first_name));
	      $this->Session->write("Member.lastName", $this->clean->stripScripts($data->ind_last_name));
	      $this->Session->write("Member.email", $this->clean->stripScripts($data->eml_address));
	      $this->Session->write("Member.location", $this->clean->stripScripts($data->adr_city . ', ' . $data->adr_state));
	      $this->Session->write("Member.loggedIn", 1);

	      $this->Session->write("Member.customerKey", $cst_key); 

	      //indicate successful login and redirect to original referrer

	   }
	   else
	   {
	      //set error message and redirect
	   }
	}
?>
```

##The SOAP Alternative

The [Avectra Wiki examples](http://wiki.avectra.com/index.php?title=XWeb:PHP) are slanted heavily in the direction of [SOAP](http://en.wikipedia.org/wiki/SOAP), however in this instance I just thought it was overkill. I figured the response was going to come back in XML, so I would just use [simplexml_load_file](simplexml_load_file) and pass in the URI. The application lives with Solaris 10 and PHP 5.2. For some reason simplexml_load_file was failing under certain boundary cases that I couldn’t quite pin down. This was an easy fix as I just used [simplexml_load_string](http://us3.php.net/manual/en/function.simplexml-load-string.php) and [file_get_contents](http://us3.php.net/manual/en/function.file-get-contents.php) to retrieve the xml response. An extra step, but it seemed to solve whatever the issue was that I was having. So, all I needed to do was make sure that I had the properly constructed URI, and that I could authenticate. Authentication is simply handled by sending in the username:password combination as part of the request.
