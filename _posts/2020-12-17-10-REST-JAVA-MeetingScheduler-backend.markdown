---
layout: post
title: Meeting Scheduler backend in Java using REST
date: 2020-12-17 18:09:50 +0300
img: MeetingScheduler/userDAO.jpg
tags: [JAVA, REST]
---

As a semestral project in a team of two, we created a backend of a meeting scheduler JAVA application using REST.
To test the application, we have written 58 unit tests and further tested it in Postman

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
Part of the project can be found here [Github](https://github.com/sciurusl/Meeting-Scheduler). There are dao objects, 
service classes, controller classes, object classes and tests.
User class:


    @Entity
    // We can't name the table User, as it is a reserved table name in some dbs, including Postgres
    @Table(name = "EAR_USER")
    @NamedQueries({
    @NamedQuery(name = "User.findByEmail", query = "SELECT u FROM User u WHERE u.username = :email"),
    @NamedQuery(name = "User.findByMeeting", query = "SELECT u.username, u.firstName, u.lastName from User u WHERE :meeting MEMBER OF u.meetings"),
    @NamedQuery(name = "User.findAll", query = "SELECT u.username, u.firstName, u.lastName from User u order by u.username"),
    @NamedQuery(name = "User.findByMeetingAll", query = "SELECT u from User u WHERE :meeting MEMBER OF u.meetings")
    })
    public class User extends AbstractEntity {
    
        @Basic(optional = false)
        @Column(nullable = false)
        private String firstName;
    
        @Basic(optional = false)
        @Column(nullable = false)
        private String lastName;
    
        @Basic(optional = false)
        @Column(nullable = false, unique = true)
        private String username;
    
        @Basic(optional = false)
        @Column(nullable = false)
        private String password;
    
        @Enumerated(EnumType.STRING)
        private Role role;
    
        @ManyToMany
        @OrderBy("organizer")
        private List<Meeting> meetings = new ArrayList<>();
    
        @OneToMany(cascade = CascadeType.REMOVE, orphanRemoval = true)
        @JoinColumn(name = "user_id")
        private List<Answer> answers;
    
        public String getFirstName() {
            return firstName;
        }
    
        public void setFirstName(String firstName) {
            this.firstName = firstName;
        }
    
        public String getLastName() {
            return lastName;
        }
    
        public void setLastName(String lastName) {
            this.lastName = lastName;
        }
    
        public String getUsername() {
            return username;
        }
    
        public void setUsername(String username) {
            this.username = username;
        }
    
        public String getPassword() {
            return password;
        }
    
        public void setPassword(String password) {
            this.password = password;
        }
    
        public void encodePassword(PasswordEncoder encoder) {
            this.password = encoder.encode(password);
        }
    
        public void erasePassword() {
            this.password = null;
        }
    
        public List<Meeting> getMeetings() {
            return meetings;
        }
    
        public void setMeetings(List<Meeting> meetings) {
            this.meetings = meetings;
        }
    
        public List<Answer> getAnswers() {
            return answers;
        }
    
        public void addMeeting(Meeting meeting) {
            Objects.requireNonNull(meeting);
            if (meetings == null) {
                this.meetings = new ArrayList<>();
            }
            meetings.add(meeting);
        }
    
        public void removeMeeting(Meeting meeting) {
            Objects.requireNonNull(meeting);
            if (meetings == null) {
                return;
            }
            meetings.removeIf(c -> Objects.equals(c.getId(), meeting.getId()));
        }
    
        public void setAnswers(List<Answer> answers) {
            this.answers = answers;
        }
    
        public Role getRole() {
            return role;
        }
    
        public void setRole(Role role) {
            this.role = role;
        }
    
        @Override
        public String toString() {
            return "User{" +
                    firstName + " " + lastName +
                    "(" + username + ")}";
        }
    }




User DAO:



    @Repository
    public class UserDao extends BaseDao<User> {
    
        public UserDao() {
            super(User.class);
        }
    
        public User findByEmail(String email) {
            try {
                return em.createNamedQuery("User.findByEmail", User.class).setParameter("email",email)
                        .getSingleResult();
            } catch (NoResultException e) {
                return null;
            }
        }
    
        public List<User> findAll() {
            return em.createNamedQuery("User.findAll", User.class).getResultList();
        }
    
    
        public List<User> findAll(Meeting meeting) {
            Objects.requireNonNull(meeting);
            return em.createNamedQuery("User.findByMeeting", User.class).setParameter("meeting", meeting)
                    .getResultList();
        }
        public List<User> findAllAll(Meeting meeting) {
            Objects.requireNonNull(meeting);
            return em.createNamedQuery("User.findByMeetingAll", User.class).setParameter("meeting", meeting)
                    .getResultList();
        }
    
    }



User controller:


    @RestController
    @RequestMapping("/rest/users")
    public class UserController {
    
        private final UserService userService;
        private final FriendsGroupService friendsGroupService;
        private final AnswerService answerService;
    
        @Autowired
        public UserController(UserService userService, FriendsGroupService friendsGroupService, AnswerService answerService) {
            this.userService = userService;
            this.friendsGroupService = friendsGroupService;
            this.answerService = answerService;
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @GetMapping(value = "/{id}", produces = MediaType.APPLICATION_JSON_VALUE)
        public User getUser(Principal principal, @PathVariable Integer id) {
            final User user = userService.find(id);
            if (user == null) {
                throw new NotFoundException("User identified by " + id + " not found");
            }
            final AuthenticationToken auth = (AuthenticationToken) principal;
            if (auth.getPrincipal().getUser().getRole() != Role.ADMIN &&
                    !auth.getPrincipal().getUser().getId().equals(id)) {
                throw new AccessDeniedException("Cannot view another user");
            }
            return user;
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @GetMapping(value = "/current", produces = MediaType.APPLICATION_JSON_VALUE)
        public User getCurrent(Principal principal) {
            final AuthenticationToken auth = (AuthenticationToken) principal;
            return auth.getPrincipal().getUser();
        }
    
        //@PreAuthorize("(!#user.isAdmin() && anonymous) || hasRole('ROLE_ADMIN')")
        @PostMapping(consumes = MediaType.APPLICATION_JSON_VALUE)
        public ResponseEntity<Void> register(@RequestBody User user) {
            userService.persist(user);
            final HttpHeaders headers = RestUtils.createLocationHeaderFromCurrentUri("/current");
            return new ResponseEntity<>(headers, HttpStatus.CREATED);
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @GetMapping(value="/{id}/meetings", produces = MediaType.APPLICATION_JSON_VALUE)
        public List<Meeting> GetMeetingsByUser(Principal principal, @PathVariable Integer id){
            User user = userService.find(id);
            if (user == null)
                throw new NotFoundException("User identified by " + id + " not found");
            else {
                final AuthenticationToken auth = (AuthenticationToken) principal;
                if (auth.getPrincipal().getUser().getRole() != Role.ADMIN &&
                        !auth.getPrincipal().getUser().getId().equals(id)) {
                    throw new AccessDeniedException("Cannot view meetings of another user");
                }
                return userService.getUserMeetings(user);
            }
        }
    
        @GetMapping(value="/{id}/friendsgroups", produces = MediaType.APPLICATION_JSON_VALUE)
        public List<FriendsGroup> GetFriendsGroupsByUser(@PathVariable Integer id){
            //User user = userService.find(id);
            //if (user == null)
            //    throw new NotFoundException("User identified by " + id + " not found");
            //else
                return friendsGroupService.findAll();
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @GetMapping(value = "/{id}/answers", produces = MediaType.APPLICATION_JSON_VALUE)
        public List<Answer> getAnswersByUser (Principal principal, @PathVariable Integer id) {
            User user = userService.find(id);
            if (user == null) {
                throw NotFoundException.create("User", id);
            }
            else {
                final AuthenticationToken auth = (AuthenticationToken) principal;
                if (auth.getPrincipal().getUser().getRole() != Role.ADMIN &&
                        !auth.getPrincipal().getUser().getId().equals(id)) {
                    throw new AccessDeniedException("Cannot view answers of another user");
                }
                return userService.getUserAnswers(user);
            }
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)
        public List<User> getAllUsers () {
            return userService.findAll();
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @PostMapping(value = "/{id}/answers", consumes = MediaType.APPLICATION_JSON_VALUE)
        public ResponseEntity<Void> createAnswerOfUser(Principal principal, @RequestBody(required = true) Answer answer, @PathVariable Integer id) {
            User user = userService.find(id);
            if (user == null) {
                throw NotFoundException.create("User", id);
            }
            final AuthenticationToken auth = (AuthenticationToken) principal;
            if (auth.getPrincipal().getUser().getRole() != Role.ADMIN &&
                    !auth.getPrincipal().getUser().getId().equals(id)) {
                throw new AccessDeniedException("Cannot create answer of another user");
            }
            answerService.persist(answer);
            //LOG.debug("Created category {}.", category);
            //final AuthenticationToken auth = (AuthenticationToken) principal;
            //final Answer answer = answerService.create(user, option, preference);
            //final User user = ((AuthenticationToken) principal).getPrincipal().getUser();
            userService.addAnswer(user, answer);
            final HttpHeaders headers = RestUtils.createLocationHeaderFromCurrentUri("/{id}", answer.getId());
            return new ResponseEntity<>(headers, HttpStatus.CREATED);
        }
    
        @PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_USER', 'ROLE_GUEST')")
        @DeleteMapping(value="/{id}/answers/{idAnswer}")
        @ResponseStatus(HttpStatus.NO_CONTENT)
        public void RemoveAnswer(Principal principal, @PathVariable Integer id, @PathVariable Integer idAnswer){
            User user = userService.find(id);
            Answer answer = answerService.find(idAnswer);
            if (answer == null)
                throw new NotFoundException("Answer identified by " + idAnswer + " not found");
            else {
                final AuthenticationToken auth = (AuthenticationToken) principal;
                if (auth.getPrincipal().getUser().getRole() != Role.ADMIN &&
                        !auth.getPrincipal().getUser().getId().equals(id)) {
                    throw new AccessDeniedException("Cannot remove answer of another user");
                }
                userService.removeAnswer(user, answer);
            }
        }
    
    }



User service:

    @Service
    public class UserService {
    
        private final UserDao dao;
        private final AnswerDao answerDao;
    
        final User currentUser = new User(); // singleton simulating logged-in user
        private final PasswordEncoder passwordEncoder;
    
        @Autowired
        public UserService(UserDao dao, PasswordEncoder passwordEncoder, AnswerDao answerDao) {
            this.dao = dao;
            this.passwordEncoder = passwordEncoder;
            this.answerDao = answerDao;
        }
    
        @Transactional
        public void persist(User user) {
            Objects.requireNonNull(user);
            user.encodePassword(passwordEncoder);
            if (user.getRole() == null) {
                user.setRole(Role.USER);
            }
            dao.persist(user);
        }
    
        @Transactional
        public void addAnswer(User user, Answer answer) {
            Objects.requireNonNull(user);
            Objects.requireNonNull(answer);
            if(user.getAnswers()==null)
                user.setAnswers(new ArrayList<>(Collections.singletonList(answer)));
            else
                user.getAnswers().add(answer);
            dao.update(user);
        }
    
        @Transactional
        public void removeAnswer(User user, Answer answer) {
            Objects.requireNonNull(user);
            Objects.requireNonNull(answer);
            answerDao.remove(answer);
        }
    
        @Transactional(readOnly = true)
        public boolean exists(String username) {
            return dao.findByEmail(username) != null;
        }
    
        @Transactional(readOnly = true)
        public User find(String email) {
            return dao.findByEmail(email);
        }
    
        @Transactional(readOnly = true)
        public User find(Integer id) {
            return dao.find(id);
        }
    
        @Transactional(readOnly = true)
        public List<User> findAll(Meeting meeting) {
            return dao.findAll(meeting);
        }
    
        @PostFilter("filterObject[0] != authentication.principal.username")
        @Transactional(readOnly = true)
        public List<User> findAll() {
            return dao.findAll();
        }
    
        /**
         * Gets the currently logged-in user's meetings.
         *
         * @return Current user's list of meetings
         */
        public List<Meeting> getCurrentUserMeetings(User user) {
            final User currentUser = SecurityUtils.getCurrentUser();
            assert currentUser != null;
            return currentUser.getMeetings();
        }
    
        public List<Meeting> getUserMeetings(User user) {
            return user.getMeetings();
        }
    
        public List<Answer> getCurrentUserAnswers(User user) {
            final User currentUser = SecurityUtils.getCurrentUser();
            assert currentUser != null;
            return currentUser.getAnswers();
        }
    
    
    
    
        public List<Answer> getUserAnswers(User user) {
            return user.getAnswers();
        }
    }
