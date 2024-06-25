---
layout: post
title:  "Fast-Emailer"
date:   2018-07-25
description: "Send mass custom emails at lightening fast speed!"
tag:
- automation 
- technology
- python
- jekyll
thumbnail: https://cdn4.iconfinder.com/data/icons/email-2/128/Email_express_mail-512.png
giscus_comments: true
---


What is this and why?
-------------------------

We felt the need for this when I was part of HINT. The main job of hosting an event on a large scale is to gather sponsors. And this requires you to reach out to them via the most underrated tool of this generation - emails. You are expected to contact lots and lots of potential sponsors, each being sent a minutely customized email asking for collaboration. To give you a fair idea, every member of the team is expected to send about 100 emails/day for about 2-3 months (depending on how effective your sponsorship drive is), which of course very few do because it's extremely monotonous and tbh boring.

I feel like breaking this into an algorithm for you:

1.  Frame the main body which would stay the same irrespective of which organization you're contacting.
2.  Insert temporary placeholders for phrases which would change depending on the organization. Eg:

> HINT would like to collaborate with {org_name}. We've been associated with {org_name} for {years} and we look forward to continuing this association.

3.  Compose a mail to the targetted organization. Paste the customized text in email body. Attach all the documents. Hit SEND.
4.  __Repeat 3 till eternity :P .__

So, step 4 is a pain. HUGE pain. Step 1-2 is fairly manual and also one-time. Step 3 is the repeated task and anything repeated can be automated. 

> **Comes in Fast-Emailer.**

It sends emails to a huge list of contacts in a matter of seconds taking care of all the minute customizations and attachments all in the comfort of your terminal.

Working
-------------------------

The code can be downloaded from [here](https://github.com/OrionStar25/Fast-Emailer). I've tried to make it modular so its easy to maintain. We would be using Gmail as a service. The script would be accessing your gmail account. 

#### Steps to let apps access your Gmail

1. Go to the [Less secure apps](https://myaccount.google.com/lesssecureapps) section of your Google Account.
2. Turn on **Allow less secure apps**. If you don't see this setting, your administrator might have turned off less secure app account access.

#### How does it work

1.  The files we have are:
    -   `app.py` - Where the main script resides
    -   `body.py` - Content of email
    -   `contacts.py` - List of contacts' emails and all their corresponding configurations
2.  The first task is to create the body of the email. Whatever phrases need to be changed according to the company need to be put in {} in `body.py`. 
    ```python
    message = "Happy birthday {Name}! You're {Age} years old now."
    ```
3.  Next, add the email ids and corresponding configurations in the form of a list in `contacts.py`.
    ```python
    Names = [
			"Niharika Shrivastava",
			"Chunnu"
			]
			
    Emails = [
			"iit2016501@iiita.ac.in",
			"chunnushrivastava@gmail.com"
			]
			
    Ages = [
		"20",
		"5"
		]
		
    Files = [
			"hb.jpg"
			]
    ```
4.  In `app.py`, our execution starts from `main()`.
    ```python
    def main():
	    get_details() 
	    print("Done!")
  
    if __name__== "__main__":
	    main() 
    ```
5.  The first job is to get the details (like your username, password, subject of email). 
    ```python
    import getpass
    
    def get_details():
        fromaddr = raw_input("Your email address: ")
        #So that no 3rd person can see the password you type into the terminal	
	    frompass = getpass.getpass(prompt='Password: ', stream=None) 
	    subject = raw_input("SUBJECT OF THE EMAIL: ")
	    
	    print("Sending.....")
    ```
6.  Then we shall loop over all the contacts and configurations we have provided and form separate emails for each of them.
    ```python
    from email.mime.application import MIMEApplication
    from email.MIMEMultipart import MIMEMultipart
    from email.MIMEText import MIMEText
    from email.MIMEBase import MIMEBase
    
    for name,age,toaddr in zip(Names,Ages,Emails):	
		msg = MIMEMultipart()
		msg['Subject'] = subject
		msg['From'] = fromaddr	
		msg['To'] = toaddr
		msg.attach(MIMEText(message.format(Name=name, Age=age)))
    ```
7.  We can also send multiple attachments to each of them.
    ```python
    for f in Files or []:
        with open(f, "rb") as fil:
		    part = MIMEApplication(fil.read(), Name=basename(f))
		part['Content-Disposition'] = 'attachment; filename="%s"' % basename(f)
	    msg.attach(part)
	    
    send_email(fromaddr, frompass, toaddr, msg, username, password) 
    ```
8. After we have formed our complete email for each individual recepient, its time to send it! We would need to establish a connection from our terminal to Gmail. For this entire process, we would use `smtplib` library.
    ```python
    import smtplib
    
    def send_email(fromaddr, frompass, toaddr, msg, username, password):
        server = smtplib.SMTP('smtp.gmail.com', 587)
	    server.starttls()
	    server.login(fromaddr, frompass)
	    text = msg.as_string()
	    server.sendmail(fromaddr, toaddr, text)
	    server.quit()
        ```
9. And thats it! 🎉🎉

#### Some possible improvements

I made this at home and it works perfectly. But after trying it out in college I was faced with the issue of proxy authorization :( I have found some fixes for that but it would either require me to use another library and not `smtplib`, or twist our proxy configuration a bit different. I'm still looking for alternatives. If you have a possible solution, do tell me! Any types of suggestions/improvements are always welcome. :D