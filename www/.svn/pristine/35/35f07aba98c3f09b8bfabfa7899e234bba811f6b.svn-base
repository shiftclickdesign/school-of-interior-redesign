<?php

   /****************************************************/
   /* CoffeeCup Software Website Search Program        */
   /* (C) 2005 CoffeeCup Software                      */
   /****************************************************/
   /* Companion Program For CoffeeCup Website Search   */
   /* Visit - http://www.coffeecup.com                 */
   /****************************************************/
   /* Constants   */
   /* Version     */ $version = '3.0';
   /* Date        */ $date = '1/05/06';
   /* Error Level */ error_reporting(E_ALL & ~E_NOTICE);

   /****************************************************/
   /* General Search Info                              */
   /****************************************************/
   /* By default, this search operates under the       */
   /* assumption that the user wants the results to    */
   /* contain all search words.  If the user wishes    */
   /* to negate this functionality, he can do this by  */
   /* using the OR operator.  Using the OR operator,   */
   /* the user can specify certain sets of words, of   */
   /* which, only one set has to have all of its       */
   /* search words found.                              */
   /*                                                  */
   /* For example, a search for:                       */
   /*                                                  */
   /* seo OR search engine optimization                */
   /*                                                  */
   /* will return results that contain seo, search     */
   /* engine optimization, or both.                    */
   /*                                                  */
   /* If the user wants to match an exact phrase, he   */
   /* can do so by putting the search string in quotes.*/
   /*                                                  */
   /* For example, a search for:                       */
   /* "Website Software" seach engine optimization     */
   /*                                                  */
   /* Will only return results that contain the        */
   /* exact phrase "Website Software" and the words    */
   /* search, engine and optimization.                 */
   /*                                                  */
   /* If a user would like to pick only results that   */
   /* don't contain a certain keyword, a user may do so*/
   /* by using the subtraction operator.               */
   /*                                                  */
   /* For example: a search for:                       */
   /* "Website software" -sitemapper                   */
   /*                                                  */
   /* Will only return pages that contain the phrase   */
   /* "Website software" that also do not contain the  */
   /* word sitemapper.                                 */
   /****************************************************/

   // Some general debugging options
   $debug = (isset($_REQUEST['debug'])) ? $_REQUEST['debug'] : $debug;
   if ($debug) error_reporting(E_ALL);
   // If debugging is enabled, print out the debugging info
   if ($debug)
   {
      switch($debug)
      {
         case 'info':
            exit(phpinfo());
         case 'version':
            exit('PHP Version: ' . phpversion() . "<br />\n" .
                 "Script Version: $version<br />\n" .
                 "Script Date: $date");
      }
   }

   // Add a message to overwrite the
   // default "no results found" message
   $custom_msg = '';

   // Fixes REQUEST_URI if they are running certain
   // version of IIS
   if(!isset($_SERVER['REQUEST_URI']))
   {
      if(isset($_SERVER['SCRIPT_NAME']))
      {
         $_SERVER['REQUEST_URI'] = $_SERVER['SCRIPT_NAME'];
      }
      else
      {
        $_SERVER['REQUEST_URI'] = $_SERVER['PHP_SELF'];
      }
      if($_SERVER['QUERY_STRING'])
      {
        $_SERVER['REQUEST_URI'] .=  '?'.$_SERVER['QUERY_STRING'];
      }
   }

   // The location of the generated search file
   $search_file = substr_replace(__FILE__, 'txt', -3);
   // The location of the generated prefs file
   $pref_file = substr_replace(__FILE__, 'xml', -3);

   // Takes care of all of the users global weight preferences.
   // The higher these numbers are set, the more the keywords that
   // are found in a given page in a given section will weigh
   // relative to the global weight of the page.
   $weights = array();
   // The global weight of the meta keys for a page
   $weights['meta_keys_weight'] = 1;
   // The global weight of the meta description for a page
   $weights['meta_desc_weight'] = 1;
   // The global weight of the user keys for a page
   $weights['user_keys_weight'] = 1;
   // The global weight of the users indiviual weight for a page
   $weights['user_weight']      = 1;
   // The global weight of the content for a page
   $weights['content_weight']   = 1;

   // The flexibility of the search.  If you set this number higher,
   // you get less, but more acurate results.  If you set this number
   // lower, you get more but less acurate results.  If you set this
   // number to a negative number, it will return all pages.
   $search_flex                 = 1;

   // The user-inputted search phrase
   $query = stripslashes($_GET['q']);
   // The current page of the search results
   $page = (!isset($_GET['p']) || (int) $_GET['p'] < 1) ? 1 : (int) $_GET['p'];



   // This class holds some general user inputted preferences for
   // the look of the search page.
   class Pref
   {
      // The html for the header
      var $htmlheader;
      // The html for the footer
      var $htmlfooter;
      // The name of the font to use
      var $htmlfontname;
      // The hex code for the background color
      var $htmlbkcolor;
      // The hex code for the text color
      var $htmltextcolor;
      // The hex code for the link color
      var $htmllinkcolor;
      // The hex code for the vlink color
      var $htmlvlinkcolor;
      // The font size to use
      var $htmlfontsize;
      // I'm not too sure about this one
      var $fontbold;
      // I'm not too sure about this one
      var $fontitalic;
      // The number of results per page
      var $resultsperpage;
      // The swf code
      var $htmlswf;
      // The date flag
      var $htmldate;
      // The def flag
      var $htmldef;
      // The rel flag
      var $htmlrel;
      // The default language
      var $htmldeflanguage;
      // The highlight color
      var $htmlhighlight;
      // The ad position
      var $adpos;
      // The ad html
      var $adhtml;
      // The font for the date
      var $htmldatefontface;
      // The font size for the date
      var $htmldatefontsize;
      // The font color for the date
      var $htmldatefontcolor;
      // The font style for the date
      var $htmldatefontstyle;
      // The font for the relevancy
      var $htmlrelevancyfontface;
      // The font size for the relevancy
      var $htmlrelevancyfontsize;
      // The font color for the relevancy
      var $htmlrelevancyfontcolor;
      // The font style for the relevancy
      var $htmlrelevancyfontstyle;
      // The font for the definition
      var $htmldefinitionfontface;
      // The font size for the definition
      var $htmldefinitionfontsize;
      // The font color for the definition
      var $htmldefinitionfontcolor;
      // The font style for the definition
      var $htmldefinitionfontstyle;

      // This function builds the prefs object given an array
      function Pref ($attr)
      {
      	 // Itterates through all the given attributes
         foreach ($attr as $name => $value)
         {
            // Check to see if we need to convert to css style
            // hex color
            if(preg_match('/^0x([\dA-F]{6})$/', $value, $match))
            {
              $this->$name = '#' . $match[1];
            }
            else
            {
              $this->$name = $value;
            }
         }

         // Setting defaults just incase
         if ($this->htmlfontname == '')
         {
            $this->htmlfontname = 'Arial';
         }
         if ($this->htmlbkcolor == '')
         {
            $this->htmlbkcolor = '#FFF';
         }
         if ($this->htmltextcolor == '')
         {
            $this->htmltextcolor = '#000';
         }
         if ($this->htmllinkcolor == '')
         {
            $this->htmllinkcolor = '#00F';
         }
         if ($this->htmlvlinkcolor == '')
         {
            $this->htmlvlinkcolor = '#F00';
         }
         if ($this->htmlfontsize == '')
         {
            $this->htmlfontsize = 13;
         }
         if ($this->resultsperpage == '')
         {
            $this->resultsperpage = 10;
         }
         if ($this->htmldatefontface == '')
         {
            $this->htmldatefontface == 'Arial';
         }
         if ($this->htmldatefontcolor == '')
         {
            $this->htmldatefontcolor = 'green';
         }
         if ($this->htmldatefontsize == '')
         {
            $this->htmldatefontsize = 13;
         }
         if ($this->htmlrelevancyfontface == '')
         {
            $this->htmlrelevancyfontface = 'Arial';
         }
         if ($this->htmlrelevancyfontcolor == '')
         {
            $this->htmlrelevancyfontcolor = 'green';
         }
         if ($this->htmlrelevancyfontsize == '')
         {
            $this->htmlrelevancyfontsize = 12;
         }
         if ($this->htmldefinitionfontface == '')
         {
            $this->htmldefinitionfontface = 'Arial';
         }
         if ($this->htmldefinitionfontcolor == '')
         {
            $this->htmldefinitionfontcolor = '0000FF';
         }
         if ($this->htmldefinitionfontsize == '')
         {
            $this->htmldefinitionfontsize = 12;
         }

         // Translate and prepare the styles for the date and url
         switch ($this->htmldatefontstyle)
         {
            case '1':
               $this->htmldatefontstyle = "\n  font-weight:bold;\n";
               break;
            case '2':
               $this->htmldatefontstyle = "\n  font-style:italic;\n";
               break;
            case '3':
               $this->htmldatefontstyle = "\n  font-weight:bold;\n  font-style:italic;\n";
               break;
            default:
               $this->htmldatefontstyle = "\n  font-weight:normal;\n  font-style:normal;\n";
         }
         // Translate and prepare the styles for the relevancy
         switch ($this->htmlrelevancyfontstyle)
         {
            case '1':
               $this->htmlrelevancyfontstyle = "\n  font-weight:bold;\n";
               break;
            case '2':
               $this->htmlrelevancyfontstyle = "\n  font-style:italic;\n";
               break;
            case '3':
               $this->htmlrelevancyfontstyle = "\n  font-weight:bold;\n  font-style:italic;\n";
               break;
            default:
               $this->htmlrelevancyfontstyle = "\n  font-weight:normal;\n  font-style:normal;\n";
         }
         // Translate and prepare the styles for the definition
         switch ($this->htmldefinitionfontstyle)
         {
            case '1':
               $this->htmldefinitionfontstyle = "\n  font-weight:bold;\n";
               break;
            case '2':
               $this->htmldefinitionfontstyle = "\n  font-style:italic;\n";
               break;
            case '3':
               $this->htmldefinitionfontstyle = "\n  font-weight:bold;\n  font-style:italic;\n";
               break;
            default:
               $this->htmldefinitionfontstyle = "\n  font-weight:normal;\n  font-style:normal;\n";
         }

         // Get the language for wiki
         switch ($this->htmldeflanguage)
         {
            case 'Deutsch':
            	$this->htmldeflanguage = 'de';
            	break;
            case 'Francais':
            	$this->htmldeflanguage = 'fr';
            	break;
            case 'Svenska':
            	$this->htmldeflanguage = 'sv';
            	break;
            case 'Nederlands':
            	$this->htmldeflanguage = 'nl';
            	break;
            case 'Polski':
            	$this->htmldeflanguage = 'pl';
            	break;
            case 'Portugues':
            	$this->htmldeflanguage = 'pt';
            	break;
            case 'Espanol':
            	$this->htmldeflanguage = 'es';
            	break;
            case 'Italiano':
            	$this->htmldeflanguage = 'it';
            	break;
            case 'Japanese':
            	$this->htmldeflanguage = 'ja';
            	break;
            default:
            	$this->htmldeflanguage = 'en';
         }


         // Just making stuff pretty for css
         $this->htmldefinitionfontsize .= 'px';
         $this->htmlrelevancyfontsize .= 'px';
         $this->htmldatefontsize .= 'px';
         $this->htmlfontsize .= 'px;';
         $this->htmlfontname .= ';';
         $this->htmlbkcolor .= ';';
         $this->htmltextcolor .= ';';
         $this->htmllinkcolor .= ';';
         $this->htmlvlinkcolor .= ';';
         $this->htmlfontsize .= ';';

         // Checking for the highlight color
         $this->htmlhighlight=(($this->htmlhighlight != '') ?
                                $this->htmlhighlight :
                                $this->htmlbkcolor) . ';';

         // If the user included a header, decode it
         if($this->htmlheader != '')
         {
            $this->htmlheader = $this->htmlheader;
         }
         // If the user included a footer, decode it
         if($this->htmlfooter != '')
         {
            $this->htmlfooter = $this->htmlfooter;
         }

         // Fixing some xml spelling errors from
         // the windows application
         $this->adhtml = ($this->addhtml) ? $this->addhtml : $this->adhtml;
         $this->adpos = ($this->addpos) ? $this->addpos : $this->adpos;
      }
   }

   // This class will be instantiated for every instance of the page
   // node found in the generated xml file.  The attributes of this
   // class correspond directly to the children of the page node.
   // This class also include various methods for searching and
   // calculating the current weight of any given page.
   class Page
   {
      // The url for the page
      var $url;
      // The date the page was last indexed
      var $indexed;
      // The meta keys for the page
      var $meta_keys;
      // The meta description of the page
      var $meta_desc;
      // The user-inputted keys for the page
      var $user_keys;
      // The user-inputted weight for the page
      var $user_weight;
      // The contents for the page
      var $content;

      // The calculated weight for the meta keys for this page
      var $meta_keys_weight;
      // The calculate weight for the meta description for this page
      var $meta_desc_weight;
      // The calculated weight for the user-inputted keys for this page
      var $user_keys_weight;
      // The calculated weight for the content for this page
      var $content_weight;

      // The total weight for this page
      var $weight;

      // This function builds the page object given an array of elements
      function Page ($attr)
      {
         foreach ($attr as $name => $value)
         {
            $this->$name = $value;
         }
         // Initialize the boled content and remove fix html entities
         $this->bolded_content = $this->content;

         // REMOVED TO COMPLY WITH EUROPEAN DATE FORMATS

         // Convert the date to a friendly format
         //$date = explode('/', $this->indexed);
         //$this->indexed = date("M d, Y", mktime(0,0,0,$date[0],$date[1],$date[2]));
      }

      // Searches and weighs this page using a given string
      function search ($string)
      {
         // Initialization of all of the local weights
         $this->meta_keys_weight = 0;
         $this->meta_desc_weight = 0;
         $this->user_keys_weight = 0;
         $this->content_weight = 0;

         // Finded quoted string for exact matches
         preg_match_all('/"([^"]*)"/', trim($string), $word_array);
         $string = preg_replace('/("[^"]*")/', '', trim($string));

         // Splits up the seach string into just words, ignoring all
         // punctuations, white-space and any other unwanted characters
         preg_match_all("/([\w'\-]*)[^\w'\-]*/",trim($string), $word_array_2);
         // Takes out the garbage
         array_pop($word_array_2[1]);

         // Merge the array from the quoted strings and the unquoted strings
         $word_array[1] = array_merge($word_array[1], $word_array_2[1]);

	 // Initialize max len for importance calculations
	 $max_len = 0;

         // Check for the OR operator
         $keys = array_keys($word_array[1], "OR");
         $count = count($keys);

         // Assigns all of the local weight multiplers for each word to be
         // multiplied against the global weight settings
         foreach ($word_array[1] as $key => $word)
         {
            // Check for the exclusion operator
            if($word{0} == '-')
            {
              $neg = true;
              $word = substr($word, 1);
            } else {
              $neg = false;
            }

            // Check for the OR operator
            if($count > 0)
            {
               // If this is our first pass,
               // set the OR array
               if(!isset($or_array))
               {
                 $or_array = array();
                 $or_array[$count] = 't';
               }
               // If the current word is OR
               // Initialize the next array
               // element
               if($word == 'OR')
               {
                  $or_array[--$count] = 't';
               }
            }

            // Calculate weights
            $content_weight = $this->get_multiplier($word, $this->content);
            $meta_keys_weight = $this->get_multiplier($word, $this->meta_keys);
            $meta_desc_weight = $this->get_multiplier($word, $this->meta_desc);
            $user_keys_weight = $this->get_multiplier($word, $this->user_keys);

            // If any word doesnt appear in one of the fields
            if($content_weight == 0 && $meta_keys_weight == 0 &&
               $meta_desc_weight == 0 && $user_keys_weight == 0)
            {
              // If the current word isn't preceeded by an
              // exlusion operator
              if(!$neg)
              {
                 // If the current search doesnt contain
                 // any OR operators
                 if(!isset($or_array))
                 {
                    // Return a weight of zero for the current page
                    return $this->weight = 0;
                 }
                 // Else, Set the current section of the OR to false
                 else
                 {
                    $or_array[$count] = 'f';
                 }
              }
            }
            // If the word does appear and the word is preceeded
            // by an exclusion operator, return a weight of zero
            // for the current page
            else if($neg)
            {
              return $this->weight = 0;
            }
            // Otherwise, calculate the current weights of the found
            // words
            else
            {
               $this->content_weight += $content_weight;
               $this->meta_keys_weight += $meta_keys_weight;
               $this->meta_desc_weight += $meta_desc_weight;
               $this->user_keys_weight += $user_keys_weight;

               // Bold all of the keywords
               $this->bolded_content = preg_replace("/\b(" . $word . ")\b/i",
                                                 '<strong>\1</strong>',
                                                 $this->bolded_content);
            }
            // Get the length of the current word
            $len = strlen($word);
            // If the length is greater than the longest word so far
            // Set the current string to the longest word
            if($len > $max_len)
            {
            	$max_len = $len;
            	$most_important = $word;
            }
         }

         // If the user is using the OR operator and none of the
         // sections found all matches, return 0 as the current
         // weight
         if(isset($or_array) && !in_array('t', $or_array))
         {
            return $this->weight = 0;
         }

         // Set the string we're looking for since we will be using it quite a bit
         $findme = "<strong>$most_important</strong>";
         // Find the position of the first occurance of this string
         $strpos = strpos(strtolower($this->bolded_content), strtolower($findme));

         // If the string can't be found in the content, simply display the
         // first 256 characters
         if($strpos === false)
         {
            $start_point = 0;
            $end_point = 256;
         }
         // If the string position is found, set a start point and an end point
         // to start clipping
         else
         {
            $start_point = $strpos - 128;
            $end_point = $strpos + 128;
         }
         // Look for a space that we can clip off before the first occurance of
         // the longest string, if one isn't found, set the start_point to the
         // first occurance of the longest string
         if(($start_point = $this->find_space($start_point, $this->bolded_content)) === false)
         {
            $start_point = $strpos;
         }
         // Look for a space that we can clip after the first occurance of
         // the longest string, if one isn't found, set the end_point to the
         // end of the first occurance of the longest string
         if(($end_point = $this->find_space($end_point, $this->bolded_content, true)) === false)
         {
            $end_point = $strpos + strlen($findme);
         }

         // Initialize the bolded content string which will be used
         // for the displayed results
         $bolded_content = '';

         // If the starting point isn't at the beginning of the content
         // add the '...' to show that the result is taken out of context
         if($start_point != 0)
         {
            $bolded_content .= '...';
         }
         // Make the clippings for the bolded content
         $bolded_content .= substr($this->bolded_content, $start_point + 1, $end_point - $start_point - 1);
         // If the ending point isn't at the end of the content
         // add the '...' to show that the result is taken out of context
         if($end_point != strlen($this->bolded_content) - 1)
         {
            $bolded_content .= '...';
         }
         $this->bolded_content = $bolded_content;

         // Returns the total weight for the current page and assigns
         // the total weight to the weight attibute
         return $this->weight = $this->get_total_weight();
      }

      // Finds a place that is alright for clipping while staying in bound
      function find_space($start, $string, $reverse = false)
      {
      	// If the reverse flag isn't set, then we move in forward motion
        if(!$reverse)
      	{
           // Set the delimeter
      	   $del = $start + 128;
      	   // Loop through all possible characters
      	   for($i = $start; $i < $del; $i++)
      	   {
      	      // Make sure we're in bounds
      	      if($i > 0)
      	      {
      	      	 // Check to see if the current character is okay for
      	      	 // a clipping point
      	         if($this->check($string[$i]) === true)
      	         {
      	            return $i;
      	         }
      	       }
      	   }
      	   // If we can't find a suitable place for clipping, we return false
       	   return false;
       	}
       	// If the reverse flag is set, the we move in revers
       	else
       	{
       	   // So we don't have to continuously calculate these
           $cnt = strlen($string);
           // Set the current delimeter
           $del = $start - 128;

           // Loop through all possible characters
       	   for($i = $start; $i > $del; $i--)
       	   {
       	      // Make sure we're in bounds
       	      if($i < $cnt)
       	      {
       	         // Check to see if the current character is okay for
       	         // a clipping point
       	         if($this->check($string[$i]) === true)
       	         {
       	            return $i;
       	         }
       	      }
       	   }
       	   // If we can't find a suitable place for clipping, we return false
       	   return false;
        }
      }

      // Check to see if a passed character is not a character that may be
      // part of a word or something else important
      function check($char)
      {
         if(ord($char) < 47 && ord($char) != 39 )
      	 {
      	    return true;
      	 }
      	 return false;
      }

      // Calculates and returns the global weight for the current page
      function get_total_weight()
      {
         global $weights;
         return ($this->meta_keys_weight * $weights['meta_keys_weight']) +
                ($this->meta_desc_weight * $weights['meta_desc_weight']) +
                ($this->user_keys_weight * $weights['user_keys_weight']) +
                ($this->content_weight * $weights['content_weight']) +
                ($this->user_weight * $weights['user_weight']);

      }

      // Calculates the multiplier for a word by counting the number
      // of times the given word appears in a section
      function get_multiplier ($needle, $haystack)
      {
         preg_match_all("/\b" . $needle . "\b/i", $haystack, $matches);
         return count($matches[0]) * strlen($needle);
      }
   }

   // Returns the current time
   function stamp ()
   {
      $time_init = explode(chr(32),microtime());
      return $time_init[1] + $time_init[0];
   }

   // Returns the amount of time that has elapsed between two times
   function elapsed ($time_start, $time_end)
   {
      return substr(($time_end - $time_start), 0, 5);
   }

   // Parses an xml file and builds an array of Page Object out of it.
   // Return the array of page objects or false if an error occurs.
   function parse_xml ($xml_file)
   {
      // Reads the xml file into a string
      $xml = file_get_contents($xml_file);

      $parser = xml_parser_create();
      // Disables casefolding for semantical purposes
      xml_parser_set_option($parser, XML_OPTION_CASE_FOLDING, 0);
      // Skips values that consist of only white space
      xml_parser_set_option($parser, XML_OPTION_SKIP_WHITE, 1);
      // If the xml fails to parse into a struct, we return false
      if(xml_parse_into_struct($parser, $xml, $values, $tags) === 0)
      {
         return false;
      }
      xml_parser_free($parser);

      // Itterates through all of the tags
      foreach ($tags as $name => $value)
      {
      	 // If we find a page node, start building the page object
         if ($name == 'page')
         {
            $nodes = $value;
            $cnt = count($nodes);
            for ($i = 0; $i < $cnt; $i += 2)
            {
              $element_array[] = build_page(array_slice($values, $nodes[$i],
                                            $nodes[$i + 1] - $nodes[$i] + 1));
            }
         }
         // If we're looking at the prefs file, load the prefs object
         else if ($name == 'coffeecupsearch')
         {
            return new Pref($values[0]['attributes']);
         }
      }
      return (count($element_array) > 0) ? $element_array : false;
   }

   // Constructs a page object using an array of corresponding nodes and values
   function build_page ($values)
   {
      $cnt = count($values);
      // Itterates through all of the nodes
      for ($i = 0; $i < $cnt; $i++)
      {
         // If an attribute array is found, add each attribute to the page array
         if(is_array($values[$i]['attributes']))
         {
            // Itterates through all of the attributes and then adds them
            // to the page array
            foreach($values[$i]['attributes'] as $name => $value)
            {
               $page[$name] = $value;
            }
         }

         // If any complete nodes are found, adds them to the page array
         if($values[$i]['type'] == 'complete')
         {
            $page[$values[$i]['tag']] = $values[$i]['value'];
         }
      }
      // Return the new page object
      return new Page($page);
   }

   // This is for sorting the page objects in descending order, based on weight.
   // Basically just your run of the mill quick sort implementation.
   function quick_sort (&$array, $left, $right)
   {
   	// hold the ends of the current array
   	$l_hold = $left;
   	$r_hold = $right;

   	// the pivot point is starts at the far left
   	$pivot = $array[$left];

   	// Itterates through every value in the current array
   	while ($left < $right)
   	{
          // So long as the weight is less than or equal to the weight of the
          // pivot object, continue on to the next value
   	  while (($array[$right]->weight <= $pivot->weight) && ($left < $right))
   	  {
   	     $right--;
   	  }
   	  // If we reach this code, and its not because we've reached the end
   	  // of the array, then we must move the current value to the other side
   	  // of the pivot
   	  if ($left != $right)
   	  {
   	     $array[$left] = $array[$right];
   	     $left++;
   	  }
   	  // Now were moving from the left to the right, so long as the weight
   	  // is greater than or equal to the pivot value, continue to the next
   	  // value
   	  while (($array[$left]->weight >= $pivot->weight) && ($left < $right))
   	  {
   	     $left++;
   	  }
   	  // If we read this code, and its not because we've reached the end
   	  // of the array, then we must move the current value to the other side
   	  // of the pivot
   	  if ($left != $right)
   	  {
   	     $array[$right] = $array[$left];
   	     $right--;
   	  }
   	}

   	// Put everything into its place
   	$array[$left] = $pivot;
   	$pivot = $left;
   	$left = $l_hold;
   	$right = $r_hold;

   	// If we still have more on the left side, continue to quick sort
   	if ($left < $pivot)
   	{
   	   quick_sort($array, $left, $pivot - 1);
   	}
   	// if we still have more on the right side continue to quick sort
   	if ($right > $pivot)
   	{
   	   quick_sort($array, $pivot + 1, $right);
   	}
   }

   // Callback function to format the definition links
   function fix_entities($matches)
   {
      global $prefs;
      return "<a target=\"_blank\" href=\"http://$prefs->htmldeflanguage.wikipedia.com/wiki/$matches[1]\">" .
             "$matches[1]</a>" . htmlentities($matches[2]);
   }

   // Get the current time
   $st = stamp();

    // Attempt to parse the preferences file
   if(($prefs = parse_xml($pref_file)) === false)
   {
      $body = "    <p class=\"error\">There seems to be a problem loading " .
              "your preferences file.</p>\n";
      $title = "Error";
   }
   // Check to make sure that the query isn't blank
   else if(trim($query) != '')
   {
      // Attempt to parse the search file
      if(($xml = parse_xml($search_file)) === false)
      {
         $body = "    <p class=\"error\">There seems to be a problem processing " .
                 "your request</p>\n";
         $title = "Error";
      }
      else
      {
         $title = 'Search results for "' . htmlentities($query) . '"';

         // Initialize the $results array
         $results = array();
         //Itterate through all of the pages
         foreach ($xml as $key => $val)
         {
            // Try to find a page that returns a weight greater than the current
            // $search_flex settings when searching for a given search string
            if($xml[$key]->search($query) > $search_flex)
            {
               $results[] = $xml[$key];
            }
         }

         // Get the number of results because we will be using it a lot
         $cnt = count($results);
         // Get the search string because we will be using it a lot
         $q = $query;
         $query = htmlentities(urldecode($query));

         // If some results were found
         if ($cnt > 0)
         {
            // Sort the results in decending order
            quick_sort($results, 0, count($results) - 1);
            // Get the result number of the first result on the current page
            $start = $page * $prefs->resultsperpage - $prefs->resultsperpage;

            // Get the heaviest weight for relevance
            $heaviest = $results[0]->weight;

            // We need to make sure that the user isn't trying to request a page
            // that is outside of this result set
            if ($start < $cnt)
            {
               // Get the results for the current page
               $page_results = array_slice($results, $start, $prefs->resultsperpage);
               if(!$prefs->btnbkd){ array_pop($page_results);
               $i = 'ICAgIDxoMz48YSBocmVmPSJodHRwOi8vd3d3LmNvZmZlZWN1cC5jb20vIiB0YXJn'.
                    'ZXQ9Il9ibGFuayI+RG93bmxvYWQgQ29mZmVlQ3VwIFNvZnR3YXJlIE5vdyAhPC9h'.
                    'PjwvaDM+CiAgICA8cD5UaGlzIHNlYXJjaCBlbmdpbmUgd2FzIGNyZWF0ZWQgdXNp'.
                    'bmcgQ29mZmVlQ3VwIFdlYnNpdGUgU2VhcmNoLjxiciAvPiBWaXNpdCA8YSBocmVm'.
                    'PSJodHRwOi8vd3d3LmNvZmZlZWN1cC5jb20iIHRhcmdldD0iX2JsYW5rIj5odHRw'.
                    'Oi8vd3d3LmNvZmZlZWN1cC5jb20vPC9hPiB0byBjaGVjayBvdXQgYWxsIG91ciBX'.
                    'ZWIgRGVzaWduIHNvZnR3YXJlIGFuZCBTZWFyY2ggRW5naW5lIFN1Ym1pc3Npb24g'.
                    'c2VydmljZXMuPGJyIC8+IDxzdHJvbmc+PGVtPlsgVGhpcyBsaW5rIGNhbiBvbmx5'.
                    'IGJlIHJlbW92ZWQgYnkgcHVyY2hhc2luZyBDb2ZmZWVDdXAgV2Vic2l0ZSBTZWFy'.
                    'Y2ggXTwvZW0+PC9zdHJvbmc+PC9wPgogICAgPHAgY2xhc3M9InVybF9pbmZvIj48'.
                    'YSB0YXJnZXQ9Il9ibGFuayIgaHJlZj0iaHR0cDovL3d3dy5jb2ZmZWVjdXAuY29t'.
                    'LyI+aHR0cDovL3d3dy5jb2ZmZWVjdXAuY29tLzwvYT4gLSA=';
               $body .= base64_decode($i).date("M d, Y").'</p>';}
               // Add the results to the body of the page
               foreach ($page_results as $page_result)
               {
                  $body .= "    <h3><a href=\"$page_result->url\">" .
                          (($page_result->title) ? $page_result->title : $page_result->url) .
                          "</a></h3>\n" .
                           "    <p>$page_result->bolded_content</p>\n" .
                           "    <p class=\"url_info\">$page_result->url" .
                           (($prefs->htmldate == 'true') ? (" - " . $page_result->indexed) : '') .
                           (($prefs->htmlrel == 'true') ?
                            (" - <span class=\"relevancy\">Relevance: " .
                            sprintf("%.2f",$page_result->weight / $heaviest * 100) . '%</span>') : '') .
                             "</p>\n";
               }

               // Check to see if we need to add navigational links to our search
         	    if ($cnt > $prefs->resultsperpage)
         	    {
         	       // Adds all previous page links
         	       for ($i = $page - 4; $i != $page; $i++)
         	       {
         	          if ($i > 0)
         	          {
         	             $navigation .= "    <a href=\"" .
         	                            htmlentities(preg_replace('/&p=\d*/','',$_SERVER['REQUEST_URI'])) .
         	                            "&amp;p=$i\">$i</a>&nbsp;\n";
         	          }
         	       }

         	       // Adds the current age number
         	       $navigation .= "    $page&nbsp;\n";

         	       // Adds all of the next page links
         	       for($i = $page + 1; $i != $page + 5; $i++)
         	       {
         	          if(($i * $prefs->resultsperpage - ($prefs->resultsperpage - 1)) <= $cnt)
         	          {
         	             $navigation .= "    <a href=\"" .
         	                            htmlentities(preg_replace('/&p=\d*/','',$_SERVER['REQUEST_URI'])) .
         	                            "&amp;p=$i\">$i</a>&nbsp;\n";
         	          }
         	          // Might as well leave the loop if weve gone out of range
         	          else
         	          {
         	    	       break;
         	          }
         	       }

         	       // Adds the "Next" and "Prev" links if needed
         	       $navigation = "<p id=\"navigation\">\n" .
         	                     (($page != 1) ? ("    <strong><a href=\"" .
         	                     htmlentities(preg_replace('/&p=\d*/','',$_SERVER['REQUEST_URI'])) .
         	                     "&amp;p=" . ($page - 1) . "\">Prev</a></strong>&nbsp;\n") : '') .
         	                     "$navigation" .
         	                     ((($page + 1) * $prefs->resultsperpage - ($prefs->resultsperpage - 1) <= $cnt) ?
         	                     ("    <strong><a href=\"" .
         	                     htmlentities(preg_replace('/&p=\d*/','',$_SERVER['REQUEST_URI'])) .
         	                     "&amp;p=" . ($page + 1) . "\">Next</a></strong>&nbsp;\n") : '') .
         	                     "  </p>";
         	    }
         	    // Get the ending time stamp
               $et = elapsed($st, stamp());

               // Preparing some infor about the current page
               $result_info = "<p id=\"result_info\">Found $cnt results for \"" .
                              (($prefs->htmldef == 'true') ?
                              (preg_replace_callback("/([\w'\-]{3,})([^\w'\-]*)/", "fix_entities", $q))
                              :
                              htmlentities($q)) .
                              "\" in $et " .
                              "seconds.<br />Displaying results " .
                              ($start + 1) . " - " . ((($start + $prefs->resultsperpage) > $cnt) ?
                              $cnt : ($start + $prefs->resultsperpage)) . "</p>";

               // If the user has included an ad
               if($prefs->adhtml != '')
               {
                  // Set the default position for an ad
                  if($prefs->adpos == '')
                  {
                     $prefs->adpos = 'right';
                  }

                  // Set the formatting for the ad
                  $prefs->adhtml = "\n\n  <!-- Start Sponsered Links -->\n  <div id=\"ads\"><h4>Sponsered Links</h4>" .
                                  "$prefs->adhtml</div>\n  <!-- End Sponsered Links-->";
                  // Adjust the styles depending on the ad position the user picked
                  // If the user wants ads on the right, float the content left and the ads
                  // right and clear it all with the nav box
                  if($prefs->adpos == 'right')
                  {
                     $adstyle = "\n\n#ads\n{\n  float:right;\n  width:20%;\n  margin:10px 50px 0 0;" .
                                "\n  padding:10px;\n  border:1px #AAA dashed;\n}\n" .
                                "\n#ads h4\n{\n  text-align:center;\n  margin:0 0 10px;\n  color:#AAA;\n}\n";
                     $results_style = "margin:0 0 50px 50px;\n  float:left;\n  width:50%;";
                     $navigation_style = "\n  clear:both;";
                  }
                  // If the user wants the ads on the left, float the ads left and the content left
                  // and clear it all with the nav box
                  else if($prefs->adpos == 'left')
                  {
                     $adstyle= "\n\n#ads\n{\n  float:left;\n  width:20%;\n  margin:10px 0 0 50px;" .
                               "\n  padding:10px;\n  border:1px #AAA dashed;\n}\n" .
                               "\n#ads h4\n{\n  text-align:center;\n  margin:0 0 10px;\n  color:#AAA;\n}\n";
                     $results_style = "margin:0 50px 50px;\n  float:left;\n  width:50%;";
                     $navigation_style = "\n  clear:both;";
                  }
                  // If the user wants the ads on the top, just display the ads as a box above the
                  // search results but below the results info
                  else if($prefs->adpos == 'top')
                  {
                     $adstyle= "\n\n#ads\n{\n  width:90%;\n  margin-left:50px;\n  padding:10px;\n" .
                               "  border:1px #AAA dashed;\n}\n" .
                               "\n#ads h4\n{\n  text-align:center;\n  margin:0 0 10px;\n  color:#AAA;\n}\n";
                     $results_style = "margin:0 50px 50px;";
                  }
                  // If the user wants the ads on the bottom, just display the ads as a box and
                  // physically move its position in the html to the bottom of the page
                  else if($prefs->adpos == 'bottom')
                  {
                     $adstyle= "\n\n#ads\n{\n  width:90%;\n  margin-left:50px;\n  padding:10px;\n" .
                               "  border:1px #AAA dashed;\n}\n" .
                               "\n#ads h4\n{\n  text-align:center;\n  margin:0 0 10px;\n  color:#AAA;\n}\n";
                     $results_style = "margin:0 50px 15px;";

                     // Move the ad to the bottom
                     $adhtml_moved = $prefs->adhtml;
                     $prefs->adhtml = '';
                 }
               }
               $definition_style = "\n\n#result_info a\n{\n  font-family:$prefs->htmldefinitionfontface;\n" .
                                   "  font-size:$prefs->htmldefinitionfontsize;\n" .
                                   "  color:$prefs->htmldefinitionfontcolor;" .
                                   $prefs->htmldefinitionfontstyle . "}";

               $date_style = "\n\n.url_info\n{\n  font-family:$prefs->htmldatefontface;\n" .
                             "  font-size:$prefs->htmldatefontsize;\n" .
                             "  color:$prefs->htmldatefontcolor;" .
                             $prefs->htmldatefontstyle . "}";

               $relevancy_style = "\n\n.relevancy\n{\n  font-family:$prefs->htmlrelevancyfontface;\n" .
                                  "  font-size:$prefs->htmlrelevancyfontsize;\n" .
                                  "  color:$prefs->htmlrelevancyfontcolor;" .
                                  $prefs->htmlrelevancyfontstyle . "}";

            }
            // The user requested an page that falls outside the current results set
            else
            {
               $body = "    <p class=\"error\"><strong>Error: </strong>The page you have requested falls outside the result set.</p>\n";
               $prefs->adhtml = '';
            }
         }
         // No matches were found
         else
         {
            $body = ($custom_msg != '') ? $custom_msg :
                   ("    <p class=\"error\">No matches found for search \"<strong><em>$query</em></strong>\"</p>\n" .
                    "    <h4>Suggestions:</h4>" .
                    "    <ul>\n" .
                    "      <li>Try making your search less specific.</li>\n" .
                    "      <li>Try using different keywords.</li>\n" .
                    "      <li>Try avoiding words that do not pertain to your search.</li>\n" .
                    "     </ul>\n");
            $prefs->adhtml = '';
         }
      }
   }
   // If the query was left blank, don't show ads
   else
   {
      $prefs->adhtml = '';
   }

   if ($results_style == '')
   {
      $results_style = "margin:0 50px 50px;";
   }

   // If the header isn't set, make one for them
   if($prefs->htmlheader == '')
     $prefs->htmlheader = <<< EOHEADER
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1 Strict//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en-US" lang="en-US">
<head>
<title>$title</title>
<style type="text/css" media="screen">
<!--
body
{
  background-color: $prefs->htmlbkcolor
  font-size:        $prefs->htmlfontsize
  font-family:      $prefs->htmlfontname
  color:            $prefs->htmltextcolor
  margin:           0;
  padding:          0;
}

a
{
  color: $prefs->htmllinkcolor
}

a:visited
{
  color: $prefs->htmlvlinkcolor
}

#results h3
{
  margin-bottom: 5px;
  line-height: 1.2em;
}

#results p
{
  margin:    0;
}

#results strong
{
   font-size: 1.1em;
   background-color:$prefs->htmlhighlight
}

#results
{
  $results_style
}

#result_info
{
  text-align: right;
  color:      #777;
  font-size:  .9em;
  margin:0 50px 20px;
}$definition_style

#search
{
  text-align: center;
  margin:     25px;
}

#navigation
{
  text-align: center;$navigation_style
}$date_style

.url_info a
{
  color:green;
  text-decoration: none;
}$relevancy_style

.error strong em
{
   background-color:$prefs->htmlbkcolor
}$adstyle

//-->
</style>
</head>
<body>

EOHEADER;

   $page = <<< EOPAGE
  <!-- Start Site Search Form -->
  <div id="search">
    $prefs->htmlswf
  </div>
  <!-- Start Site Search Form -->

  <!-- Start Results Info -->
  $result_info
  <!-- End Results Info -->$prefs->adhtml

  <!-- Start Results -->
  <div id="results">
$body  </div>
  <!-- End Results -->$adhtml_moved

  <!-- Search Page Navigation -->
  $navigation
  <!-- End Search Page Navigation -->

EOPAGE;

   // If the footer isn't set, make one for them
   if($prefs->htmlfooter == '')
   {
     $prefs->htmlfooter = "</body>\n</html>";
   }

   // Give the loader time to display
   sleep(2);

   // Display the results
   echo $prefs->htmlheader . "\n" . $page . "\n" . $prefs->htmlfooter;

?>
