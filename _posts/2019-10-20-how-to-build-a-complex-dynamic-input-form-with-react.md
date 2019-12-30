---
layout: post
title: How to Build a Complex Dynamic Input Form with React
date: 2019-10-20 14:24
summary: Building form has never been complicated than before...
categories: react frontend software-development tech
tags: software-development front-end tech
---

[![Image from Gyazo](https://i.gyazo.com/2db07c1bcf30e071d3402e7d76668523.gif)](https://gyazo.com/2db07c1bcf30e071d3402e7d76668523)

Making forms in 2019 has never been more complicated than before. React has made building forms and UI's easier but creating a complex form input that creates a seamless experience to the users can be tricky and challenging. I was recently assigned to create an on-call support application.  Each user in the team will be able to view his/her on-call schedule, where team leads, and admins can form groups with admins and agents. 

One of the challenges was to create a new team registration form.  Therefore, I would like to share about how I designed and build this dynamic input field so that you can use this concept in your next React Project. 


## The Problem
The form contains admin, agent input field, and team name. I need to create a team form where the user can add admin and agents easily. 

### These are the rules of the form:
A team needs to have an admin. A user can be in multiple groups, and the user can be an admin in one group and an agent on the other. 

There will be a list of users in the database somewhere coming from the backend. I need to design the form to mitigate any invalid form submission to the backend. Moreover, I need also to design the form to guide users in submitting the form in the right way, without explicitly instructing them how to do so. 

If a user is selected as an admin, that user should not show up again in the agent input field. For instance, if the client selects "*John Doe*"  as an admin in Team A, then John Doe will not appear in the agents' select input. 

_[![Image from Gyazo](https://i.gyazo.com/1883a27a6a4c98529961c2be2039420d.gif)](https://gyazo.com/1883a27a6a4c98529961c2be2039420d)_

Users can dynamically add or delete any users in the admin, and agents select the input field in real-time without corrupting the state of the application.


## The Intuition
Since there are a set of users' arrays coming in from the backend, I decided to use a select input field in the admin and users' field. Having a set of input fields, I will use input type text for doing team names and have real-time validation on that.

How to make sure if the users are not duplicate in the admins and agents' input section?
Since I want to minimize the amount of fetch call to the server, the only time I get the user is when the component is mounted. Therefore, I need to keep a copy from the original users' array from the server so that any addition and deletion on the user lists will not mutate the original data coming from the server. This user pool will be used to implement any operations in the form input field. This user pool will use in both agents and admins input fields. 

Each time the client chooses a user from the admin's input selection, that user is deleted from the pool. This way, I can mitigate duplication in both the select input field for admin and user without corrupting the state of the application. 

## Order of Attack
I usually start by drawing the diagram of components, either on a paper or with any mock wireframe technology that you want to use. Then, I identify the <a href="https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0" target="_blank">dummy and intelligent component</a>.

I will start from the most straightforward presentational component and work backward to the container component.

## The Execution
Create Presentation Component
The presentation layer will be something like this:

{% highlight javascript %}
const TeamForm = () => {
    return (
        <>
            <label htmlFor="admins">Team Name</label>
            <input type="text" value={team.name} onChange={handleChange} name={'name'} placeholder="Team Name" />
            <div>
                <label htmlFor="admins">Admins</label>
                <select name={"admins"} value={''} onChange={handleChange}>
                    {usernameList.map(({id, username}) =>
                        <option key={uuid.v4()} value={id}>{username}</option>
                    )}
                </select>
            </div>
            <div>
                <label htmlFor="agents">Agents</label>
                <select name={"agents"} value={''} onChange={handleChange}>
                    {usernameList.map(({id,username}) =>
                        <option key={uuid.v4()} value={id}>{username}</option>
                    )}
                </select>
            </div>
            <button onClick={handleSubmit}>Submit</button>
        </>
    )
}
{% endhighlight %}

In this code, I used a single select input form to see how the code looks on the browser.

<img src="{{site.baseurl}}/images/how-to-build-a-complex-dynamic-input-form-with-react/PresentationForm.png" alt="PresentationForm">

Add the required state and event handler in the container component
Once I see the presentation component, I can see the require the state to store in the container component. 
Then, we think about the representation of data to pass down to the presentation component:

Do the users and admin need to be an array to create multiple dynamic inputs? 
What kind of event handler do I need to have?


{% highlight javascript %}
const TeamForm = ({
    handleSubmit,
    handleChange,
    handleDeleteClick,
    team,
    usernameList,
}) => {
    return (
        <>
            <label htmlFor="admins">Team Name</label>
            <input type="text" value={team.name} onChange={handleChange} name={'name'} placeholder="Team Name" />
            <div>
                <label htmlFor="admins">Admins</label>
                {team.admins && team.admins.map(admin => {
                    return (<div key={admin.id}>
                        <span> {admin.username} </span>
                        <button onClick={handleDeleteClick('admins', admin.id)}> - </button>
                    </div>)
                })}
                <select name={"admins"} value={''} onChange={handleChange}>
                    {usernameList.map(({id, username}) =>
                        <option key={uuid.v4()} value={id}>{username}</option>
                    )}
                </select>
            </div>
            
            <div>
                <label htmlFor="agents">Agents</label>
                {team.agents && team.agents.map(agent => {
                    return (<div key={agent.id}>
                        <span> {agent.username} </span>
                        <button onClick={handleDeleteClick('agents', agent.id)} > - </button>
                    </div>)
                })}
                <select name={"agents"} value={''} onChange={handleChange}>
                    {usernameList.map(({id,username}) =>
                        <option key={uuid.v4()} value={id}>{username}</option>
                    )}
                </select>
            </div>
            <button onClick={handleSubmit}>Submit</button>
        </>
    )
}
{% endhighlight %}

In this case, I need to create `handleChange` event in the container component to receive the data that the user-triggered in input form; `handleDelete` to received delete messages from the child component, and another event handler to receive a news when the client click the `Submit` button. I need to create these three handlers in Container Component.

Tackle down container component
This is the meat of our form logic. This is where you want to put all the implementation from the intuition that I talk about earlier for a dynamic add and delete select input form located. 

Handle Change:
{% highlight javascript %}
    const handleChange = (event) => {
        const { name, value } = event.target;
        // if it is selected, automatically add to the team and create a new selection input
        // this can combine because the how the state is design in the component
        // name and value representing the property of the state
        if (name === 'admins' || name === 'agents') {
            const newUserObj = users.find(user => user.id === Number(value));
            console.log('what is newUserObj', newUserObj);
            console.log(name);
            setTeam(prevTeam => ({
                ...prevTeam,
                [name]: prevTeam[name].concat(newUserObj),
            }))
        }

        // changing team name
        else if (name === 'name') {
            setTeam(prevTeam => ({
                ...prevTeam,
                [name]: value,
            }));
        }
    }
{% endhighlight %}

Handle Delete:
{% highlight javascript %}
 const handleDeleteClick = (authority, id) => (event) => {
        setTeam(prevTeam => ({
            ...prevTeam,
            [authority]: prevTeam[authority].filter(user => user.id !== id),
        }));
    }
{% endhighlight %}

Next, to prevent duplication in form select from admin and agent, create a clone buffer of the users from the list, called `userList`, when the component is initially mounted. This way, each time when there are an `onChange` events on one of the input fields, it can filter out the `userList` from the team objects before rendering to `TeamForm.jsx` (preventing duplication in both selection input). Then, create a team object that will serve as a temporary state in the presentation component, `TeamForm.jsx`. 

 {% highlight javascript %}
 export const getUsersNotInTeam = (usersList, team) => {
    const { admins = [], agents = [] } = team;
    return usersList.filter(user => {
        return !(admins.find(u => u.id === user.id) ||
            agents.find(u => u.id === user.id));
    });
}
 {% endhighlight %}

Put it together
This is the full code for ManageTeamPage.jsx and TeamForm.jsx: 
<iframe height="400px" width="100%" src="https://repl.it/repls/SlateblueGrimyMath?lite=true" scrolling="no" frameborder="no" allowtransparency="true" allowfullscreen="true" sandbox="allow-forms allow-pointer-lock allow-popups allow-same-origin allow-scripts allow-modals"></iframe>

There you have it! A basic run through from my thought process on how to create a dynamic input form to React. I didn't talk about the validation part and will leave that to the next topic. If you have any questions, please feel free to comment on the section below. I hope this can help you tackle down your next project. 

This is the full code for the project in <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/dynamic-input-tutorial" target="_blank">GitHub</a>.



