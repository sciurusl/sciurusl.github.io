---
layout: post
title: Meeting Scheduler backend in Java using REST
date: 2020-12-17 18:09:50 +0300
img: MeetingScheduler/userDAO.jpg
tags: [JAVA, REST]
---

As a semestral project in a team of two, we created a backend of a meeting scheduler JAVA application using REST.
To test the application, we have written 58 unit tests and even tested it in 

The system description:
The system is similar to Doodle, but has more functions. The
organizer offers several options for a meeting in terms of time slots and possible
places. Each poll participant decides how much the offer is good for him/her (e.g.
in terms of 0-10 points) and can fill in a remark.

## Functionality
- Creation of meetings
    - places
    - time slots
    - max. number of people
- Meeting evaluation by participant
    - scale of 1-10
    - leave a comment
- Finalize (creator only)
    - mark final decision

### List of roles
- creator of meeting - can later edit places, time slots etc.
- participant

### Overall description

### Product perspective
Database stores the following information:
* Meetings:
This includes a time stamp of when a meeting was created, its participants,
available options to choose from and a final decision, given one was issued
yet.


* Users:
It includes first name, last name, his email, password and meetings he was
part of.

### Meeting class and characteristics
User should be able to do following actions:
- Create meeting
- Add an answer to a meeting he has been invited to participate in
  - Vote for multiple options
  - Set preference from 1-10
- View current status for a meeting he has participated in

Creator of the meeting should also have the following actions:
* Add/remove available times
* Add/remove available places
* Add a comment after a decision has been taken
* Edit meeting later
* Delete his meeting

### Project Structure

![model]({{site.baseurl}}/images/pages/MeetingScheduler/model.jpg)

ER model:
![ER model]({{site.baseurl}}/images/pages/MeetingScheduler/ERModel.jpg)

### Code example
User class:
![user]({{site.baseurl}}/images/pages/MeetingScheduler/user.jpg)

User DAO:
![user DAO]({{site.baseurl}}/images/pages/MeetingScheduler/userDAO.jpg)

User controller:
![user Controller]({{site.baseurl}}/images/pages/MeetingScheduler/userController.jpg)

User service:
![user Service]({{site.baseurl}}/images/pages/MeetingScheduler/userService.jpg)