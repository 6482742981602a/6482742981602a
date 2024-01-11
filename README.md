#30



Programming

Creating a Credit Card Validation Class With PHP

￼

Mitchell Harper and David RusikApril 17, 2002

Share

Although online payment options such as PayPal have become extremely popular in the last couple of years, the majority of online stores still use some sort of merchant system to accept credit card payments from their Websites. Before you actually encrypt your customer’s credit card numbers to a database or forward them to a merchant server, it’s a good idea to implement your own credit card validation routine.

PauseNext

Unmute

Current Time 0:07

/

Duration 2:00

Loaded: 36.53%

 

Fullscreen

In this article we’re going to work through the development of a PHP class that stores the details of a credit card and validates its number using the Mod 10 algorithm. To implement the class we’ll create in this article, you should have access to an Apache web server running PHP 4.1.0 or later.

Credit Card Validation

What do we actually mean when we say "validate a credit card number"? Quite simply it means that we run a credit card number through a special algorithm known as the Mod 10 algorithm.

This algorithm processes some simple numerical data validation routines against the number, and the result of this algorithm can be used to determine whether or not a credit card number is valid. There are several different types of credit cards that one can use to make a purchase, however they can all be validated using the Mod 10 algorithm.

As well as passing the Mod 10 algorithm, a credit card number must also pass several different formatting rules. A list of these rules for each of the six most popular credit cards is shown below:

￼

mastercard: Must have a prefix of 51 to 55, and must be 16 digits in length.

Visa: Must have a prefix of 4, and must be either 13 or 16 digits in length.

American Express: Must have a prefix of 34 or 37, and must be 15 digits in length.

Diners Club: Must have a prefix of 300 to 305, 36, or 38, and must be 14 digits in length.

Discover: Must have a prefix of 6011, and must be 16 digits in length.

JCB: Must have a prefix of 3, 1800, or 2131, and must be either 15 or 16 digits in length.

As mentioned earlier, in this article we will create a PHP class that will hold the details of a credit card number and expose a function that indicates whether or not the number of that credit card is valid (i.e. whether it passed the Mod 10 algorithm or not). Before we create that class however, let’s look at how the Mod 10 algorithm works.

The Mod 10 Algorithm

There are three steps that the Mod 10 algorithm takes to determine whether or not a credit card number is valid. We will use the valid credit card number 378282246310005 to demonstrate these steps:

ADVERTISEMENT

ADVERTISEMENT

Step One

The number is reversed and the value of every second digit is doubled, starting with the digit in second place:

378282246310005

becomes…

500013642282873

￼

and the value of every second digit is doubled:

5 0 0 0 1 3 6 4 2 2 8 2 8 7 3

x2 x2 x2 x2 x2 x2 x2

——————————————-

0 0 6 8 4 4 14

ADVERTISEMENT

ADVERTISEMENT

Step Two

The values of the numbers that resulted from multiplying every second digit by two are added together (i.e. in our example above, multiplying the 7 by two resulted in 14, which is 1 + 4 = 5). The result of these additions is added to the value of every digit that was not multiplied (i.e. the first digit, the third, the fifth, etc):

5 + (0) + 0 + (0) + 1 + (6) + 6 + (8) + 2 + (4) + 8 + (4) + 8 + (1 + 4) + 3

￼

= 60

Step Three

When a modulus operation is applied to the result of step two, the remainder must equal 0 in order for the number to pass the Mod 10 algorithm. The modulus operator simply returns the remainder of a division, for example:

10 MOD 5 = 0 (5 goes into 10 two times and has a remainder of 0)

ADVERTISEMENT

ADVERTISEMENT

20 MOD 6 = 2 (6 goes into 20 three times and has a remainder of 2)

43 MOD 4 = 3 (4 goes into 43 ten times and has a remainder of 3)

So for our test credit card number 378282246310005, we apply a modulus of 10 to the result from step two, like this:

60 MOD 10 = 0

The modulus operation returns 0, indicating that the credit card number is valid.

￼

Now that we understand the Mod 10 algorithm, it’s really quite easy to create our own version to validate credit card numbers with PHP. Let’s create our credit card class now.

Creating the CCreditCard Class

Let’s now create a PHP class that we can use to store and validate the details of a credit card. Our class will be able to hold the cardholder’s name, the card type (mastercard, visa, etc), the card number, and the expiry month and date.

Create a new PHP file called class.creditcard.php. As we walk through the following steps, copy-paste each piece of code shown to the file and save it.

We start of by defining several card type constants. These values will be used to represent the type of card that our class will be validating:

ADVERTISEMENT

ADVERTISEMENT

<?php  
 
define("CARD_TYPE_MC", 0);  
define("CARD_TYPE_VS", 1);  
define("CARD_TYPE_AX", 2);  
define("CARD_TYPE_DC", 3);  
define("CARD_TYPE_DS", 4);  
define("CARD_TYPE_JC", 5);

Next, we have our class declaration. Our class is called CCreditCard. Note that there is an extra ‘C’ at the front of the class name intentionally: it’s a common programming practice to prefix the name of a class with ‘C’ to in fact indicate that it is a class.

We also define five member variables, which will be used internally to hold the credit card’s name, type, number, expiry month and year:

class CCreditCard  
{  
// Class Members  
var $__ccName = '';  
var $__ccType = '';  
var $__ccNum = '';  
var $__ccExpM = 0;  
var $__ccExpY = 0;

Next we have our class’ custom constructor. A constructor is a function that has the same names as the class in which it exists. It returns no value. It is special in the sense that it is automatically executed whenever we create a new instance of that class.

Whenever we want to create a new instance of our CCreditCard class, we must explicitly pass in five arguments to its constructor: the cardholder’s name, card type, number, and expiry date. Because we have created our own custom constructor (PHP implements a default constructor that accepts no arguments if we don’t explicitly create one), we must pass in values for each of these five arguments every time we instantiate the class. If we omit them, PHP will raise an error.

// Constructor   
function CCreditCard($name, $type, $num, $expm, $expy)   
{

If the value of the $name variable passed into the constructor is empty, then we use the die() function to terminate the instantiation of our class and output an error message telling the user that they must pass a name to the constructor:

// Set member variables   
if(!empty($name))   
{   
$this->__ccName = $name;   
}   
else   
{   
die('Must pass name to constructor');   
}

Our CCreditCard class is flexible: it accepts several different ways to specify the type of card that is being stored. For example, if we want to add the details of a mastercard to a new instance of our CCreditCard class, then we could pass in the following values for the $type variable of the constructor: "mc", "mastercard", "m", or "1".

We make sure that a valid card type has been passed in, and set the value of our classes $__ccType variable to one of the constant card type values that we defined earlier:

// Make sure card type is valid   
switch(strtolower($type))   
{   
  case 'mc':   
  case 'mastercard':   
  case 'm':   
  case '1':   
    $this->__ccType = CARD_TYPE_MC;   
    break;   
  case 'vs':   
  case 'visa':   
  case 'v':   
  case '2':   
    $this->__ccType = CARD_TYPE_VS;   
    break;   
  case 'ax':   
  case 'american express':   
  case 'a':   
  case '3':   
    $this->__ccType = CARD_TYPE_AX;   
    break;   
  case 'dc':   
  case 'diners club':   
  case '4':   
    $this->__ccType = CARD_TYPE_DC;   
    break;   
  case 'ds':   
  case 'discover':   
  case '5':   
    $this->__ccType = CARD_TYPE_DS;   
    break;   
  case 'jc':   
  case 'jcb':   
  case '6':   
    $this->__ccType = CARD_TYPE_JC;   
    break;   
  default:   
    die('Invalid type ' . $type . ' passed to constructor');   
}

If an invalid card type is passed in, then the default branch of our switch statement will be called, resulting in our script terminating with the die() function.

We can take advantage of PHP’s built-in support for regular expressions by using the ereg_replace function to strip out all non-numeric characters from the credit card number:

// Don't check the number yet,    
// just kill all non numerics    
if(!empty($num))    
{    
  $cardNumber = ereg_replace("[^0-9]", "", $num);    
   
  // Make sure the card number isnt empty    
  if(!empty($cardNumber))    
  {    
    $this->__ccNum = $cardNumber;    
  }    
  else    
  {    
    die('Must pass number to constructor');    
  }    
}    
else    
{    
  die('Must pass number to constructor');    
}

We finish off our CCreditCard constructor by making sure that both the expiry month and year are valid, numerical values:

if(!is_numeric($expm) || $expm < 1 || $expm > 12)    
{    
  die('Invalid expiry month of ' . $expm . ' passed to constructor');    
}    
else    
{    
  $this->__ccExpM = $expm;    
}    
   
// Get the current year    
$currentYear = date('Y');    
settype($currentYear, 'integer');    
   
if(!is_numeric($expy) || $expy < $currentYear || $expy    
> $currentYear + 10)    
{    
  die('Invalid expiry year of ' . $expy . ' passed to constructor');    
}    
else    
{    
  $this->__ccExpY = $expy;    
}    
}

In our CCreditCard class, the only way to set the values of the credit card’s details is through the constructor. To retrieve the values of our class-specific variables ($__ccName, $__ccType, etc), we create several functions, like this:

function Name()    
{    
  return $this->__ccName;    
}    
   
function Type()    
{    
  switch($this->__ccType)    
    {    
    case CARD_TYPE_MC:    
      return 'mastercard [1]';    
      break;    
    case CARD_TYPE_VS:    
      return 'Visa [2]';    
      break;    
    case CARD_TYPE_AX:    
      return 'Amex [3]';    
      break;    
    case CARD_TYPE_DC:    
      return 'Diners Club [4]';    
      break;    
    case CARD_TYPE_DS:    
      return 'Discover [5]';    
      break;    
    case CARD_TYPE_JC:    
      return 'JCB [6]';    
      break;    
    default:    
      return 'Unknown [-1]';    
  }    
}    
   
function Number()    
{    
  return $this->__ccNum;    
}    
   
function ExpiryMonth()    
{    
  return $this->__ccExpM;    
}    
   
function ExpiryYear()    
{    
  return $this->__ccExpY;    
}

These functions allow us to retrieve the values of the variables contained within our class. For example, if I created an instance of our CCreditCard class called $cc1, then I could retrieve its expiration month using $cc1->ExpiryMonth().

A common function when working with credit cards is displaying the details that you’ve captured from that user back to them as a confirmation. For example, if the user entered a credit card number of 4111111111111111, then you might want to only show part of the number to them, such as 4111111111111xxxx. Our CCreditCard class contains a function called SafeNumber, which accepts two arguments. The first is the character to mask the digits with, and the second is the number of digits to mask (from the right):

function SafeNumber($char = 'x', $numToHide = 4)     
{     
  // Return only part of the number     
  if($numToHide < 4)     
  {     
    $numToHide = 4;     
  }     
    
  if($numToHide > 10)     
  {     
    $numToHide = 10;     
  }     
    
  $cardNumber = $this->__ccNum;     
  $cardNumber = substr($cardNumber, 0, strlen($cardNumber) - $numToHide);     
    
  for($i = 0; $i < $numToHide; $i++)     
  {     
    $cardNumber .= $char;     
  }     
    
  return $cardNumber;     
}

If we had an instance of our CCreditCard class called $cc1 and the credit card number stored in this class was 4242424242424242, then we could mask the last 6 digits like this: echo $cc1->SafeNumber('x', 6).

The last function contained in our CCreditCard class is called IsValid, and implements the Mod 10 algorithm against the credit card number of our class, returning true/false.

It starts of by setting two variables ($validFormat and $passCheck) to false:

function IsValid()     
{     
  // Not valid by default     
  $validFormat = false;     
  $passCheck = false;

Next we make sure that the credit card number is formatted correctly. We use PHP’s ereg function to do this. The regular expression that must be matched is different for each card:

// Is the number in the correct format?     
switch($this->__ccType)     
{     
  case CARD_TYPE_MC:     
    $validFormat = ereg("^5[1-5][0-9]{14}$", $this->__ccNum);     
    break;     
case CARD_TYPE_VS:     
    $validFormat = ereg("^4[0-9]{12}([0-9]{3})?$", $this->__ccNum);     
    break;     
case CARD_TYPE_AX:     
    $validFormat = ereg("^3[47][0-9]{13}$", $this->__ccNum);     
    break;     
case CARD_TYPE_DC:     
    $validFormat = ereg("^3(0[0-5]|[68][0-9])[0-9]{11}$", $this->__ccNum);     
    break;     
case CARD_TYPE_DS:     
    $validFormat = ereg("^6011[0-9]{12}$", $this->__ccNum);     
    break;     
case CARD_TYPE_JC:     
    $validFormat = ereg("^(3[0-9]{4}|2131|1800)[0-9]{11}$", $this->__ccNum);     
    break;     
  default:     
  // Should never be executed     
  $validFormat = false;     
}

At this point, $validFormat will be true (ereg returns true/false) if the credit card number is in the correct format, and false if it’s not.

We now implement a PHP version of the Mod 10 algorithm, using exactly the same steps that we described earlier:

// Is the number valid?      
$cardNumber = strrev($this->__ccNum);      
$numSum = 0;      
     
for($i = 0; $i < strlen($cardNumber); $i++)      
{      
  $currentNum = substr($cardNumber, $i, 1);      
     
// Double every second digit      
if($i % 2 == 1)      
{      
  $currentNum *= 2;      
}      
     
// Add digits of 2-digit numbers together      
if($currentNum > 9)      
{      
  $firstNum = $currentNum % 10;      
  $secondNum = ($currentNum - $firstNum) / 10;      
  $currentNum = $firstNum + $secondNum;      
}      
     
$numSum += $currentNum;      
}

The $numSum variable will contain the sum of all of the variables from step two of the Mod 10 algorithm, which we described earlier. PHP’s symbol for the modulus operator is ‘%‘, so we assign true/false to the $passCheck variable, depending on whether or not $numSum has a modulus of zero:

// If the total has no remainder it's OK      
$passCheck = ($numSum % 10 == 0);

If both $validFormat and $passCheck are true, then we return true, to indicate that the card number is valid. If not, we return false, to indicate that either the card number was in an incorrect format, or if failed the Mod 10 check:

  if($validFormat && $passCheck) return true;      
  else return false;      
 }      
}      
?>

And that’s all there is to our CCreditCard class! Let’s now look at a simple validation example using HTML forms, PHP, and an instance of our CCreditCard class.

Using our CCreditCard Class

Create a new file called testcc.php and save it in the same directory as the class.creditcard.php file. Enter the following code into testcc.php:

<?php include('class.creditcard.php'); ?>      
<?php      
if(!isset($submit))      
{      
?>      
     
  <h2>Validate Credit Card</h2>      
  <form name="frmCC" action="testcc.php" method="post">      
     
  Cardholders name: <input type="text" name="ccName"><br>      
  Card number: <input type="text" name="ccNum"><br>      
  Card type: <select name="ccType">      
  <option value="1">mastercard</option>      
  <option value="2">Visa</option>      
  <option value="3">Amex</option>      
  <option value="4">Diners</option>      
  <option value="5">Discover</option>      
  <option value="6">JCB</option>      
  </select><br>      
     
  Expiry Date: <select name="ccExpM">       
     
  <?php      
     
    for($i = 1; $i < 13; $i++)      
    { echo '<option>' . $i . '</option>'; }      
     
  ?>       
     
  </select>      
     
  <select name="ccExpY">      
     
  <?php      
     
    for($i = 2002; $i < 2013; $i++)      
    { echo '<option>' . $i . '</option>'; }      
     
  ?>       
     
  </select><br><br>      
     
  <input type="submit" name="submit" value="Validate">      
  </form>      
     
  <?      
     
  }      
  else      
  {      
  // Check if the card is valid      
  $cc = new CCreditCard($ccName, $ccType, $ccNum, $ccExpM, $ccExpY);      
     
  ?>      
     
  <h2>Validation Results</h2>      
  <b>Name: </b><?=$cc->Name(); ?><br>      
  <b>Number: </b><?=$cc->SafeNumber('x', 6); ?><br>      
  <b>Type: </b><?=$cc->Type(); ?><br>      
  <b>Expires: </b><?=$cc->ExpiryMonth() . '/' .      
  $cc->ExpiryYear(); ?><br><br>      
     
  <?php      
      
    echo '<font color="blue" size="2"><b>';       
     
    if($cc->IsValid())      
    echo 'VALID CARD';      
    else      
    echo 'INVALID CARD';      
     
    echo '</b></font>';      
  }      
?>

Run the script in your browser and see what happens…

Here are two screen shots from my browser. The first one shows the HTML form, and the second shows the output once the form is submitted:

￼
Entering the card details

￼
The results of the card validation

Conclusion

In this article we’ve seen how we can take advantage of PHP’s object oriented features (most notably classes) to create a credit card storage and validation class. We went through the components of this class in detail, and we finished off by creating a test script in which we instantiated our CCreditCard class and validated a sample card number.

If you’re thinking of setting up an eCommerce site that will process/store visitors’ credit card details, then you should take the class we’ve just made and customize it to suit your needs. You might want to add other functions to it to compare CCreditCard objects, format the card’s details into an XML string, encrypt the card’s details to a database, or even process the payment in real time.

Share This Article

ADVERTISEMENT

ADVERTISEMENT

￼

Mitchell Harper and David Rusik

Mitchell and David joined forces to bring you this tutorial.

Up Next

￼

Form Validation with PHPIain Tench, Matthew Setter

￼

Building a Credit Card Form Custom Element with PolymerPankaj Parashar

￼

Creating an Animated Valentine’s Day Card with Snap.svgIvaylo Gerchev

￼

Creating an iOS Credit Card Reader with XamarinMark Trinder

￼

PayPal Credit Card Tokenization in MagentoChris Nanninga

￼

SitePoint Code Challenge: CSS – Flip A CardSarah Hawk

Stuff we do

Premium

Newsletters

Forums

About

Our story

Terms of use

Privacy policy

Corporate memberships

Contact

Contact us

FAQ

Publish your book with us

Write an article for us

Advertise

Connect

© 2000 – 2024 SitePoint Pty. Ltd.

This site is protected by reCAPTCHA and the Google Privacy Policy and Terms of Service apply.


