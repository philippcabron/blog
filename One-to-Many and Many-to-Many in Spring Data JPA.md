#One-to-Many and Many-to-Many in Spring Data JPA

_Or better: Why I struggled to build my first own REST-API_

Building a REST API is one of the basics in backend software developing, but I still hadn’t dealt with it yet. Thus, I was that guy who wasn’t on that party everyone talks about - I needed to change this.
Luckily, there are tons of opportunities to include a REST API in a project. My project is a portfolio website – also a thing fancy developers should have. Therefore, I started to read into that topic and structure my tech stack (Angular, Spring, Azure SQL Server).
For my backend I’ve chosen Spring, if you are familiar with it then you are probably already a fan of Baeldung – a perfect website to learn about the Spring Framework. They helped me a lot and without their guides and tutorials I wouldn’t be where I am now (my thesis was written in Spring Boot). So, Kudos to you guys!

But now to my REST API...

The data model is quite simple: a _Project_ entity which can have several _Skills_ that are used in it. Additionally, there is an _Employment_ entity with details about the company and role. 
 

![Simplified Class Diagram](https://raw.githubusercontent.com/philippcabron/images/master/class%20diagram.drawio.png)

#☝ --> 🖐
Between _Employment_ and _Project_ is a **One-to-Many** relationship. Nothing special – I just followed the tutorial on [Baeldung](https://www.baeldung.com/hibernate-one-to-many). For these two entities I went with a bidirectional relationship, means that I am able to access _Projects_ from _Employments_ and _Employments_ from _Projects_. For better understanding, I added my model classes in a shortened version below. Annotations like @Getter, &#64Setter and @Data are from Lombok - a nice little library to reduce boilerplate code.
 
![Employment Class](https://raw.githubusercontent.com/philippcabron/images/master/Employment.png)


In line 17 of _Employment Class_, you can see the _@OneToMany_ annotation for field _projects_. The attribute _mappedBy_ aims on field _employment_ of my _Project_ class. I could leave it like that, and it would work, but I want a bidirectional relationship. Therefore, the field _employment_ of my _Project_ class needs a _@ManyToOne_ annotation as you can see below (_Project Class_).

![Project Class](https://raw.githubusercontent.com/philippcabron/images/master/Project.png) 

Until now it wasn’t a big deal except for one little recursion issue on my _Employment_ entity, because of the fact, that my Json response of an _Employment_ object includes the related _Projects_ and these having an _Employment_ object each with their _Projects_ and so on. I tried to fix that issue by adding _@JsonIgnore_ to field _projects_. However, by adding this annotation, a field or method is being “ignored by introspection-based serialization and deserialization functionality. That is, it should not be consider a ‘getter’, ‘setter’ or ‘creator’.” – according to the [fasterxml documentation](https://fasterxml.github.io/jackson-annotations/javadoc/2.5/com/fasterxml/jackson/annotation/JsonIgnore.html). Important to notice, that the whole _projects_ field is ignored, so that we’re getting an _Employment_ object in our response without any sign of the related _projects_. Oh dear, did I struggle with this annotation. At this point I was going in circles, cause my assumption was that I found the right annotation and was missing something else. After what felt like an eternity, I finally read about _@JsonIgnoreProperties_. Essentially the same but instead of ignoring the whole object, it is ignoring just individual fields/properties. In my case, it is ignoring the _Employment_ field in my _Project_ objects.
The recursion is gone and the One-to-Many relationship works fine. My next step was to implement a Many-to-Many relationship. So, I used my advanced Google techniques to find a good way to do it. 😎
 

![Google Search](https://raw.githubusercontent.com/philippcabron/images/master/giphy.gif)


#🖐 --> 🖐
There are several different approaches to handle the **Many-to-Many** relationship between _Project_ and _Skill_. According to the [tutorial on Baeldung](https://www.baeldung.com/jpa-many-to-many) you need to create a join entity class, if you want to add attributes to your relationship. Therefore, I added one for my join table _ProjectSkills_ (see “_Simplified Class Diagram_”), because of the _usageInPercentage_ attribute. 
A join table primary key consists of foreign keys, a so-called composite key. In Spring you need to consider this for your join entity class. In figure _ProjectSkills Class_ you might notice _ProjectSkillsKey_ field with the _@EmbeddedId_ annotation - that is our composite key. For the sake of completeness, I also attached the _ProjectSkillsKey Class_ below. It simply combines our both foreign keys.
 
![ProjectSkills Class](https://raw.githubusercontent.com/philippcabron/images/master/ProjectSkills.png)

![ProjectSkillsKey Class](https://raw.githubusercontent.com/philippcabron/images/master/ProjectSkillsKey.png)

Having a join entity class dissolves the Many-to-Many relationship and converts it into two separate One-to-Many relationships. Except for one small difference, this structure is identical to the relationship between _Employment_ and _Project_. The difference: Another annotation is needed - _@MapsId_. It is used to indicate which the primary key of the annotated attribute is.
After implementing the _ProjectSkills_ class, all needed relationships were portrayed. Thus, I was done with my entities. After all, I know, this is neither new nor exciting stuff, but it still gave me a good feeling to do it and write about it. 

Thank you for your attention! Additionally, I am new to writing blogs, especially in English. Tips, tricks or language corrections are welcome! Many thanks in advance! ❤️

And don’t forget to add _@JsonIgnoreProperties_ to avoid recursions! 
