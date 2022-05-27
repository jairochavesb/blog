---
layout: default
title: Jairo's Blog
---
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
* {
  box-sizing: border-box;
}

#myInput {
  background-image: url('/css/searchicon.png');
  background-position: 10px 12px;
  background-repeat: no-repeat;
  width: 100%;
  font-size: 16px;
  padding: 12px 20px 12px 40px;
  border: 1px solid #ddd;
  margin-bottom: 12px;
}

#myUL {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

#myUL li a {
  border: 1px solid #ddd;
  margin-top: -1px; /* Prevent double borders */
  background-color: #ffffff; /*#f6f6f6;*/
  padding: 12px;
  text-decoration: none;
  font-size: 18px;
  color: black;
  display: block
}

#myUL li a:hover:not(.header) {
  background-color: #eee;
}
</style>
</head>

<h2>Archive</h2>

<input type="text" id="myInput" onkeyup="myFunction()" placeholder="Search post titles..." title="Type in a name">

<ul id="myUL">
  <li><a href="https://jairochavesb.github.io/blog/posts/2022/05/26/malware_vaccines_part1.html">Malware Vaccines Using Infection Markers. Part 1.</a></li>
  <li><a href="https://jairochavesb.github.io/blog/posts/2022/04/11/golang-nmclui.html">Golang - nmclui</a></li>
  <li><a href="https://jairochavesb.github.io/blog/posts/2022/02/28/hermetic-wiper.html">Analysis - Hermetic Wiper</a></li>
  <li><a href="https://jairochavesb.github.io/blog/posts/2022/02/16/elearnsec-ecmap-review.html">eLearnSecurity - Certified Malware Analysis Professional</a></li>
  <li><a href="https://jairochavesb.github.io/blog/posts/2021/12/cracking-challenge-by-disip.html">Crackmes.one - Cracking challenge by Disip</a></li>
  <li><a href="https://jairochavesb.github.io/blog/posts/2021/12/process-injection-with-go.html">Process Injection with Go</a></li>
  <li><a href="https://jairochavesb.github.io/blog/posts/2021/12/call-windows-api-with-go.html">Calling Windows APIs with Go</a></li>
</ul>

<script>
function myFunction() {
    var input, filter, ul, li, a, i, txtValue;
    input = document.getElementById("myInput");
    filter = input.value.toUpperCase();
    ul = document.getElementById("myUL");
    li = ul.getElementsByTagName("li");
    for (i = 0; i < li.length; i++) {
        a = li[i].getElementsByTagName("a")[0];
        txtValue = a.textContent || a.innerText;
        if (txtValue.toUpperCase().indexOf(filter) > -1) {
            li[i].style.display = "";
        } else {
            li[i].style.display = "none";
        }
    }
}
</script>

