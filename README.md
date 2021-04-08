# Results Web App ([Demo](https://results-web.2mm.io/))

## Demo Credentials

**Admin:** admin@admin.com  
**Password:** secret

### Site 1 users: 

**Manager:** manager@user.com  
**Password:** secret

**User:** user@user.com  
**Password:** secret

---
## Introduction

The Repora public document is to aim at clarifying information in a public domain.

---
## [Using our API](#using-our-api)

The benefit of building an API is putting all the complicated business logic and handling inside an interface so that you can have multiple clients using the same endpoint to achieve the same task. E.g in our case here, Repora mobile app using a bundle of API endpoints to *create, update and delete* appointment, which can be used by public as long as they have the correct authorization. 

### [Create Appointment](create-appointment)

The only prerequisite for creating the appointment will be your organisation ID, and technician ID. For organisation ID(site_id), it's easy to retrieve as your login will reveal that. 

Once you call login API and retrieve your ```access_token```, you can call the below query for find out your own information: 

Get ```site_id``` from your current login info: 
```gql
query CurrentLogin{
  me{
    site_id
    first_name
    last_name
    full_name
    email
  }
}
```

Get ```site_id``` from your current site info: 
```gql
query CurrentSite{
  current_site {
    id
    business_name
    abn
    site_type {
      name
    }
  }
}
```

Get ```user_id``` from users query:
```gql
query users{
  users(first: 10, page: 1) {
    data {
      id
      full_name
      email
    }
  }
}
```
This Graphql query will return you a list of users(technician) from your organisation. 

**Important** Please make sure your login as a site manager otherwise you cannot see other users except your current login.

Once you has ```site_id``` and ```user_id``` ready, you can start calling Appointment APIs. 

---
#### [Create Appointment](create-appointment-code)

```gql
mutation createAppointment{
  createAppointment(input: {
    site_id: 1
    user_id: 4
    schedule_date: "2021-04-22 13:30:00"
    appointment_status: 1
    confirmation_status: 1
    billing_status: 1
  }){
    id
    user_id
    schedule_date
		appointment_status{
      id
      name
    }
    confirmation_status{
      id
      name
    }
    billing_status{
      id
      name
    }
  }
}
```
In the above query, ```appointment_status, confirmation_status and billing_status``` are nullable parameter which means you do not need to include them in your query if you are not having information of them. 

```appointment_status, confirmation_status and billing_status``` can be query by: 
```gql
query systemDefaults {
  appointment_status {
    id
    name
  }
  confirmation_status {
    id
    name
  }
  billing_status {
    id
    name
  }
}
```

Simple version without ```appointment_status, confirmation_status and billing_status```.
```gql
mutation createAppointment{
  createAppointment(input: {
    site_id: 1
    user_id: 4
    schedule_date: "2021-04-22 13:30:00"
  }){
    id
    user_id
    schedule_date
		appointment_status{
      id
      name
    }
    confirmation_status{
      id
      name
    }
    billing_status{
      id
      name
    }
  }
}
```
> For other available fields in the query, refer to the Graphql-Playground docs

Once you create an appointment, you can retrieve the ```appointment_id``` from the response of the above query if API return you successfully. Then you can **update** or **delete** appointment

Update appointment:
```gql
mutation updateAppointment{
  updateAppointment(input: {
    id: 1
    user_id: 3
    schedule_date: "2021-02-07 13:35:00"
    billing_status: 1
    appointment_status: 1
    confirmation_status: 1
    billing_status: 1
  }){
    id
    schedule_date
    appointment_status{
      id
      name
    }
    confirmation_status{
      id
      name
    }
    billing_status{
      id
      name
    }
  }
}
```

Delete appointment:
```gql
mutation deleteAppointment{
  deleteAppointment(id: 1){
    id
    schedule_date
    appointment_status{
      name
    }
    confirmation_status{
      name
    }
    billing_status{
      name
    }
  }
}
```

## License

MIT: [https://mit-license.org/](https://mit-license.org/)
