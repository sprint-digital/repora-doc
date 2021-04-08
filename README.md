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

Results web app is the backend server for the Repora Apps. It aims to support a scalable infrastructure for the SaaS scenarios(Multi-tenancy). 

### [Terminology](terminology)
**Site**
> Representing a tenant or organisation. Everything that with a ```site_id``` indicates it should be an entity or config separated from other tenant or organisation

**Appointment**
> An appointment would be the first entity for a business instance between the organisation and its customers. Everything with appointment_id is an underlying entities of an appointment e.g. ```report, appointment_contact, appointment_user, appointment_address```.

**Report**
> The main entity of this app. It contains multiple structures and details under those structures. And this would be the 3rd level that spin off from **Site**. Any entities underlying ```report``` with ```report_id``` will belong to a report. And since at this level, a ```report``` is unique to a tenant, and to an appointment, therefore the underlying entities will no longer point to **Site** or **Appointment**, but **Report**.

**Report Structure** vs **Report Structure Item**

> In the database design, ```report_structure``` representing the a structure within a report. A report can have multiple structure, such as "Main building", "Granny Flat". 

!IMPORTANT, for all the sections ```StructureDetails, Termite History, Previous Treatment, Measurements, Inspection Findings, Treatment Recommendations```, they are all under ```report_structure```.

> As for ```report_structure_item```, it points to the item in design ```Structure Details``` sections.

For the above mentioned sections:

| UI Sections              | Database Table        |
|          ---:            |         :---          |
|Termite History           | report_history        |
|Previous Treatment        | report_past_treatment |
|Structure Details         | report_structure_item |
|Measurements              | report_measurements   |
|Inspection Findings       | report_inspection     |
|Treatment Recommendations | report_treatment      |

---
### Treatment, Product, Measurement

The relationship of treatment, product and measurement is the core of the report cost calculation at the end. 

Please refer to the DB design for clarification: [Results DB Structure](https://app.sqldbm.com/MySQL/Edit/p136243/#).
For its login, ask Joel. 

Please refer to this field heading matrix for accurate data
[Treatment, Product, Measurement](https://docs.google.com/spreadsheets/d/1SFnlUdrEs72jkquXYdhEU2MJWe3WVh0NC1wP003JXt8/edit?ts=6019f89c#gid=0)

---
## [Tech choices](tech-choice)

### Graphql

Based on the complexity and the depths of the database structure, we decided to change the **Restful** patterns to **Graphql**.

In this repo, [Lighthouse-PHP](https://lighthouse-php.com/) is the laravel graphql library we use. It natively supports a lot of the Eloquents making quering and mutation super easy. 

Alternatively, you can override the native way on query, mutation by creating your own. They are all sitting in ```app/Graphql```.

For Graphql development, ```Graphql-Playground``` would be very helpful. 

For local environment, once your server is up, you can open in [http://localhost/graphql-playground](http://localhost/graphql-playground).

For staging environment, [https://results.2mm.io/graphql-playground](https://results.2mm.io/graphql-playground).


### Media Library

For all the image handling, we use [Spatie Media Library](https://spatie.be/docs/laravel-medialibrary/v9/introduction). Be aware that this library is using **Poly Morph**, therefore all the image references are stored in one table.

Meanwhile, the storage will be using S3 for staging and production, while in local, ```minio``` is used. It would be recommended that when creating the ```laradock``` cluster for Repora backend, please include ```minio```. If you are using the ```setup.sh``` script, you can using ```./setup.sh -rmo``` to spin up your instance. 

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

MIT: [http://anthony.mit-license.org](http://anthony.mit-license.org)
