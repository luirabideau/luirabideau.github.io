---
layout: essay
type: essay
title: "Assignment 2 Reflection"
# All dates must be YYYY-MM-DD format!
date: 2023-12-15
published: true
labels:
  - ITM
  - Assignment 2
  - Reflection
---
Well... you shall <a href="https://youtu.be/Ngfovy91qRM">receive</a>.
Oh no. Lui doesn't know how to format CSS.
<p style = "color:white">
/* Created by Lui Rabideau on 12/5/2023 */
/* Incorporated into the design from W3schools: W3.CSS 4.15 December 2020 by Jan Egil and Borge Refsnes */
/* UHM ITM352 Assignment 2 */
const { error } = require('console');
const express = require('express');
const app = express();
const querystring = require('querystring');

// USERDATA FILE STUFF FROM LAB13 FILE IO 

// global variable to store IR5 stuff (users logged in)
let userLoggedin = {};


const fs = require('fs');
const { type } = require('os');
let user_reg_data = {};
let user_data_filename = __dirname + '/user_data.json';
// if the user data file exists, read it and parse it
if (fs.existsSync(user_data_filename)) {
    // let user_reg_data = require('./user_data.json');
    let user_reg_data_JSON = fs.readFileSync(user_data_filename, 'utf-8');
    user_reg_data = JSON.parse(user_reg_data_JSON);
} else {
    console.log(`Error! ${user_data_filename} does not exist!`);
}

// END of lab13
 
// express decodes URL encoded data in the body
app.use(express.urlencoded({ extended: true }));
// Routing 

// Handling invoice requests 
/* 
app.get('/invoice.html', function (request, response, next) {
   console.log('attempted ' + request.method + ' to ' + request.path);
   response.redirect(`./login.html`);
});*/

// monitor all requests
app.all('*', function (request, response, next) {
   console.log(request.method + ' to ' + request.path);
   next();
});

// process a route for the products.json stuff
const products = require(__dirname + "/products.json");
// making a products array within the server
app.get('/products.js', function(request, response, next){
   // the response will be js
   response.type('.js');

   // turning stuff into a string
   let products_str = `let products = ${JSON.stringify(products)}`;
   // sends the string
   response.send(products_str);
});

// process a route for the professionals.json stuff
professionals = require(__dirname + "/professionals.json");
// making a professionals array within the server
app.get('/professionals.js', function(request, response, next){
   // the response will be js
   response.type('.js');
   // turning stuff into a string
   let professionals_str = `let professionals = ${JSON.stringify(professionals)}`;
   // sends the string
   response.send(professionals_str);
});

// making a userInfo array within the server
app.get('/userLoggedin.js', function(request, response, next){
   // the response will be js
   response.type('.js');
   // turning stuff into a string
   let userLoggedin_str = `let userLoggedin = ${JSON.stringify(userLoggedin)}`;
   // sends the string
   response.send(userLoggedin_str);
});

let qs;
// process purchase request (validate quantities, check quantity available)
app.post('/process_purchase', function (request, response, next){
   // get the quantity data
   // validate the quantity data
   let noInput = 0; // defined a variable to keep track of how many products have 0 or empty quanities
   let errors = {}; // assume no errors to begin with
   for (let i in products){ 
      let q = request.body[`quantity${i}`];
      // validate the quantities
      if (validateQuantities(q, false) == false){
         errors[`error_quantity${i}`] = validateQuantities(q, true);
      }
      // the serverside part of IR3
      // Check if there is enough avaliable, if not send error
      if(q > products[i].aval){
         errors[`error_quantity${i}`] = `There are not ${q} avaliable.`
      }
      // did the user select any item
      if((request.body[`quantity${i}`]) == 0 || (request.body[`quantity${i}`]) === ''){
         // if the quantity is 0 or empty, adds 1 to the noInput variable
         noInput++;
      } 
   };
   // this checks if all the product quantities were either 0 or empty
   if(noInput == products.length){
      response.redirect(`./products.html?NoInput`); //sends the error back to the page
   }else{
      // if valid, send info to invoice
      qs = querystring.stringify(request.body);
      if (Object.entries(errors).length === 0) {
      // part of IR1 
      //remove quantities from products.aval
         for(let i in products){
            products[i].aval -= request.body[`quantity${i}`];
         }
         // sending to invoice
         response.redirect(`./login.html`)
      }
      // if invalid, redisplay products but with errors
      else{ 
         response.redirect(`./products.html?${qs}&${querystring.stringify(errors)}`);
      }
   }
});


// route all other GET requests to files in public 
app.use(express.static(__dirname + '/public'));
// start server
app.listen(8080, () => console.log(`listening on port 8080`));

// the isNonNegInt function with name changes
function validateQuantities(quantities, returnErrors){
   // assume no errors at first
   errors = [];
   if(quantities === ''){
      quantities = 0;
   };
   // Check if string is a number value
   if(Number(quantities) != quantities){ 
      errors.push('Not a number!');};
   // Check if it is non-negative
   if(quantities < 0){ 
      errors.push('Negative value!');}; 
   // Check that it is an integer
   if(parseInt(quantities) != quantities){ 
      errors.push('Not an integer!');}; 
   // if true will return errors, if not returns nothing
   return returnErrors ? errors : (errors.length == 0);
};


// ASSIGNMENT 2

app.post('/login', function (request, response) {
   // Process login form POST and redirect to logged in page if ok, back to login page if not
   let the_username = request.body.username.toLowerCase();
   let the_password = request.body.password;
   // check if username is in user_data
   console.log(`${qs}`);
   console.log(`${the_username} is working`);
   if(the_username == ''|| the_password == ''){
      response.redirect(`./login.html?Error=NoInput`);
   }
   if(typeof user_reg_data[the_username] !== 'undefined'){
      // check if the password matches the password in user_reg_data
      if(user_reg_data[the_username].password === the_password){
         console.log(`${the_username} is logged in!`);
            // add new logged in user
            userLoggedin[the_username] = true;
            response.redirect(`./invoice.html?${qs}&email=${the_username}&name=${user_reg_data[the_username].name}&numUsersLoggin=${Object.entries(userLoggedin).length}`);
      } else { // else the password does not match
         response.redirect(`./login.html?Error=InvalidPassword`);
      }
   } else { // else the user does not exist 
      response.redirect(`./login.html?Error=InvalidUsername`);
   }
});

app.post('/register', function (request, response) {
// process a simple register form
//make a new user
let username = request.body.username.toLowerCase();
user_reg_data[username] = {};
fullName = request.body.firstname + ' ' + request.body.lastname;
user_reg_data[username].name = fullName; 
user_reg_data[username].password = request.body.password;
user_reg_data[username].username = request.body.username;
pass1 = request.body.password;
pass2 = request.body.conpassword;
if(pass1 !== pass2){
   response.redirect(`./register.html?Error=PassUnmatch`);
}
// add it to the user_data.json
fs.writeFileSync(user_data_filename, JSON.stringify(user_reg_data));
//checking if the there is anything inputted
if(typeof user_reg_data[username] !== 'undefined' || typeof user_reg_data[username].password !== 'undefined' || typeof user_reg_data[username].username !== 'undefined' || user_reg_data[username] !== ' ' || user_reg_data[username].password !== ' ' || user_reg_data[username].username !== ' '){  
   // check if usrname is taken
  if(typeof user_reg_data[username] !== 'undefined') {
   response.redirect(`./register.html?Error=UsedEmail`)
   }
   // ChatGPT code that sees if there is an @ and . present in the right order
   // prompt used: simple email address validator function javascript
   function validateEmail(email) {
      // Check if the email contains "@" and "."
      const atIndex = email.indexOf('@');
      const dotIndex = email.lastIndexOf('.');
      // Ensure "@" comes before "."
      return atIndex !== -1 && dotIndex !== -1 && atIndex < dotIndex;
    }
    if (validateEmail(user_reg_data[username].username)) {
      // add new logged in user, place above the redirect
      userLoggedin[username] = true; 
      response.redirect(`./invoice.html?${qs}&email=${username}&name=${user_reg_data[username].name}&numUsersLoggin=${Object.entries(userLoggedin).length}`); 
    } else {
      response.redirect(`./register.html?Error=InvalidEmail`);
    }
} else {
   response.redirect(`./register.html?Error=NoInput`);
}
});

app.post('/thankYou', function (request, response) {
   let shipLine0 = request.body.shipName
   let shipLine1 = request.body.shipAd1;
   let shipLine2 = request.body.shipAd2;
   response.redirect(`./thankyouPage.html?shipLine0=${shipLine0}&shipLine1=${shipLine1}&shipLine2=${shipLine2}`)
});
</p>
