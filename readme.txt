Calciu Alexandru, 331CC

task 1:

it took me a while to find out what i had to do but i figured nmap would help,
luckily there weren't a lot of machines on the local network. i found out
that the ip of the server ends with .36 but still went through ip a s and nmap
when logging in to work on the other tasks.

after running the web tunnel script i went on the site and had a frustrating
time looking for something in the page sources that would lead me to the good
url or some hidden buttons or stuff like that. it took some time but i found
that commented <a> tag that had the url for the real register
(/auth/register_real_one or sth like that). i made an account and thought that
was that but when i logged in i saw no flag and was disappointed. i was
even more disappointed when i had to dig around through html again
just to find that the flag was written in white font over a white
background. dirty and unnecessary move.

task 2:

i knew js was awful but jquery is just on another level, man. the task file
said xss so i went to every input field and put a <script> tag with a console
log to see where i can work my magic. turns out, it was the one in the messages
page. i accepted nyan's friend request and thought about how to approach this.
i figured that to become friends with boss i had to make him accept my request
somehow or sth like that. this didn't work via direct messages so i had to find
another route. i stumbled upon the page source by accident and saw that huge
comment block with advice and suddenly the road became more clear.

i first tried some console logs to see that my script gets executed as nyan,
which did happen. then i tried to call some functions i found in main.js
when looking for the flag in task 1. i received errors when doing that
and it took me some time to figure out that it was because i didn't wrap
my script with that $(function() {}); jquery magic. this was a wonderful
discovery.

i then started thinking about how to get to boss, and my solution was
to redirect nyan's form to point to boss instead of me(i saw that on
my page, the form had an action that ended in 6, which also happened
to be nyan's profile number(which i discovered by playing with urls)).
i discovered that my profile id was 9 and manipulated nyan's form
using jquery. after that, i took control of his textarea and entered
another script with a console log with the name of the logged in user.
then i used the submit line that was also in the comments and that was
pretty much the script. it worked, i got boss in the console, but as
i mentioned before, when i tried to do something with acceptFriend
it crashed. after fixing that, i also found out that i had to escape the
</script> tag in the second script because it would act as an end tag
for the main script which i found a little funny. i fixed that and
everything worked.

the full script is here, unfiltered:

//===BEGIN CODE===

<script>$(function() {
console.log(

window.authUserName + " at " + window.location);

if (window.authUserName === 'nyan') {

    $('form').attr('action', '/inside/message/send/1');
    $('textarea').html("sunt aici");
    
    $('textarea').text("sunt si aici");
    
    $('textarea').text("<script> $(function() {console.log('sunt sefu ' + window.authUserName); acceptFriend(9); console.log($('.acceptFriend'));}) <\/script>");
    $('form').submit()
}
});
</script>

//===END CODE=====

task 3:

after finishing task 2, which took some effort, i had genuinely no idea
where to look for clues for task 3. i didn't know what it wanted from me so
i went ahead and looked at the page source, tried downloading the website
recursively for some reason. after some failed attempts which led to nothing,
i used my expert level detective skills to come to the conclusion that
those two messages from boss must have a purpose. one was the flag, so the
other had to have some sort of a clue.

the only useful information in that message was something about a certain
backup.sh script, the task.txt file said something about downloading, so i
tried slamming backup.sh in the website url. surprisingly, it actually worked
and it was really nice. i read the script and saw that it did some archive
black magic and saw that there was a final archive made somewhere that
contained two concatenated archives. i tried some reasonable url combinations
to get the archive until i finally found it. i know i could have made a script
to go through the dates and do wget on those generated urls but i saw that
it's not that big of a range and was too lazy to try complicated stuff.

after downloading the archive i had some trouble opening it, more precisely
getting the flag out of it. after some google searches, i found out that i
have to use the i flag for tar to dearchive iteratively or sth like that.
i tried tar ixvf on the archive and sure enough, it worked. i really
appreciated that the flag was in plaintext now and not hidden behind
a cipher or some other shenanigan just to make the task needlessly
tedious.

task 4:

i hate sql and didn't like the sql injection lab. i started out by sticking
' everywhere i could until sql started showing signs of weakness. the location
was surprising, the profile url of all places. i started out with one column and
worked my way up until sql stopped crying. the magic number was 8 apparently.
this is the command i used at first, adding numbers to the list progressively:

' union select 1 from information_schema.tables --%20

the %20 is a code for space, it has to be there or sql has an aneurysm.
after some inspection of the page i found out where most of the numbers
went. 3 was for the profile name, 6 for the description, and the rest are useless.
because it brings luck, i used the third field for the next commands.
after finding out this exciting information i copied and pasted the group
concat magic from the lab to get the table_schema, since i didn't know the
schema unfortunately. i used distinct because without it it was a lot of
information_schema spam. this is the command:

' union select 1,2,group_concat(distinct(table_schema)),4,5,6,7,8 from information_schema.tables --%20

the output had 3 things, one of them which i found more interesting: web_62
i figured that must be a good contender for the table_schema, so i went ahead
and used it going forward. as per the lab, i had to find out the names of
the tables, which i did using the following command:

' union select 1,2,group_concat(distinct(table_name)),4,5,6,7,8 from information_schema.tables where table_schema='web_62' --%20

one of the names in the output was flags20707 so it was pretty obvious where
to look. as per the lab, i then set out to find the names of the columns
using the following command:

' union select 1,2,group_concat(distinct(column_name)),4,5,6,7,8 from information_schema.columns where table_name='flags20707' --%20

one of the columns was named zaflag and again the way forward was pretty
obvious. the last command just involved getting what was on that column
in the flags20707 table, and it looked something like this:

' union select 1,2,zaflag,4,5,6,7,8 from flags20707 --%20

and in the end there was a flag. what an awful task.

task 5:

i stopped the web tunnel and used the trustworthy tcpdump because if i see the
word traffic i know that using tcpdump is the point of the task. initially,
for reasons beyond my humble understanding, there were a lot of packages
flying around and i didn't know where to look and gave up for the day.


when i came back to it, all that spam magically went away and the output
was more readable. i saw that someone was sending udp stuff to a port on
the dev machine and so i went ahead and listened on that port using netcat
(nc -l -u <the ip address of the machine> <the port shown in the output from
tcpdump>). the output was full of nyans and a string of random characters
that ended in "=", which i was pavlov-ed to recognize as encoded using
base64. i took that text, put it in a file, used base64 -d on that file
and that was the flag. i found this step to be a little unnecessary but
whatever.