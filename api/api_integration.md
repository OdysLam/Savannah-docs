## API Integration

While Savannah provides a number of built-in data sources, your community will likey exist and be active in some systems that it doesn't know about. In these cases Savannah provides an `API Integration` Source type that will enable you to push Member and Conversation data directly into your Savannah community.

## Creating an API Source

All data imported into Savannah is associated with a `Source` object, and API integrations are no different. In order to add your external system to Savannah you must first create an `API Integration` Source, in the same way as you create other sources.

![Add API Integration Source](./AddSourceMenu.png) ![Add API Integration Source](./AddSourceForm.png)

Your `Source` will be given a unique authorization token that you will use when making calls to the Savannah API. Because each token is associated with a `Source`, any data added to Savannah using that token will be associated with that `Source`.

![Source with Token](./APISourceToken.png)

## API Basics

### Authorization

API calls will need to be authorized using the API token of your `Source`. This is done by placing it in an HTTP request header:

```
Authorization: token f32fde77-ebbb-4799-94f7-065846da88bf
```

### Timestamps

Timestamps will always use the format `YYYY-MM-DDThh:mm[:ss[.uuuuuu]][+HH:MM|-HH:MM|Z]`
### Origin IDs

Savannah's API doesn't expose internal IDs, instead it uses unique identifiers from your external system to lookup internal records. These are referred to as the `origin_id` of the record.

You will need to provide an ID for each `Identity` and `Conversation` that is unique to your system. These IDs should be unique to your integration source, but do not need to be globally unique for your community. For example, you can have an `Identity` with the id `testuser` in your integration source, and also a user with the same identity from a Slack or Github source. 

Merging multiple identities into the same `Member` can be done in Savannah the same way it is for non-API sources.

### Repeating API calls

Savannah's APIs are designed to be idempotent. That means that calling the same API with the same data multiple times will not create duplicate data.

As long as you use the same `origin_id` each time, subsequent calls will only update the existing data associated with that ID.

### Create rather than fail

Whenever an API call references an `origin_id` for a record that is not already in Savannah, a new record for that ID will be created with minimal information rather than causing the API call to fail with an error.

You will still need to make additional API calls to fill in the missing data for those newly created records, but you will not be required to do that beforehand.

## Identities

An `Identity` is what associates a `Member` of your community with a `Source`. Members can (and usually do) have more than one `Identity`.

The `Identity` API endpoint is:

```
https://savannahhq.com/api/v1/identity/
```

![Create an Identity](./IdentityAPI.png)

## Conversations

The `Conversation` API endpoint is:

```
https://savannahhq.com/api/v1/conversation/
```

![Create a Conversation](./ConversationsAPI.png)

When creating a `Conversation` you will need to provide certain fields in the form of their respetive `origin_id`. These records will be created in Savannah if they haven't already been added.

* **speaker**: The id of the person who started the conversation
* **channel**: The id of the channel (however your source defines them) that the conversation happened in
* **participants**: A list of ids for the people who were tagged, replied to, or otherwise were a participant in this conversation.

Savannah will create a `Connection` record between the `speaker` and any `particpants` in the conversation.