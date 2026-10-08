# SnapShare Scaling Plan

## Assumptions

SnapShare has 10,000,000 registered users.

10% of users are active each day:

10,000,000 × 10% = 1,000,000 daily active users.

Each active user:

- uploads 1 photo per day
- views 50 feed pages per day

Each photo is about 2 MB.

Each photo also gets a thumbnail of about 50 KB.

For rough estimates, one day is about 100,000 seconds.

Peak traffic is estimated as 5 times the average.

## Traffic Estimates

### Uploads per day

1,000,000 active users × 1 upload per day = 1,000,000 uploads per day.

### Average uploads per second

1,000,000 ÷ 100,000 = about 10 uploads per second.

### Peak uploads per second

10 × 5 = about 50 uploads per second.

### Feed views per day

1,000,000 active users × 50 feed views = 50,000,000 feed views per day.

### Average feed views per second

50,000,000 ÷ 100,000 = about 500 feed views per second.

### Peak feed views per second

500 × 5 = about 2,500 feed views per second.

## Storage Estimate

Each uploaded photo uses:

2 MB for the original photo

plus about 50 KB for the thumbnail.

50 KB is about 0.05 MB.

Total storage per photo is about:

2 MB + 0.05 MB = 2.05 MB.

Daily storage:

1,000,000 photos × 2.05 MB = 2,050,000 MB per day.

2,050,000 MB = about 2,050 GB per day.

2,050 GB = about 2.05 TB per day.

Yearly storage:

2.05 TB × 365 = about 748 TB per year.

Therefore, SnapShare needs roughly 748 TB of new photo storage per year.

## Read-Heavy or Write-Heavy

SnapShare is read-heavy.

There are about 500 feed views per second on average compared with about 10 photo uploads per second.

At peak, there are about 2,500 feed views per second compared with about 50 uploads per second.

Because there are many more reads than writes, the system should use caching, a CDN and database read replicas to reduce the load on the main database and improve feed speed.

## Why Photos Should Not Be Stored in the Database

The actual photo files should not be stored directly inside the database because they are large and would make the database much bigger and harder to manage.

The database should store information about each photo, such as the user id, caption, upload time and the location of the photo file.

The photo files themselves should be stored in object storage such as Amazon S3 or another object storage service.

Object storage is designed to store large files cheaply and efficiently.

## Architecture Diagram

    Users / Clients
          |
          | static files and photos
          v
    +------------------+
    |       CDN        |
    +------------------+
          |
          | API requests
          v
    +------------------+
    |  Load Balancer   |
    +------------------+
          |
      +---+---+
      |       |
      v       v
    +-------+ +-------+
    | App 1 | | App 2 |
    +-------+ +-------+
      |   |      |   |
      |   |      |   |
      |   +------+   |
      |      |       |
      |      v       |
      | +-----------+|
      | |   Cache   ||
      | |  Redis    ||
      | +-----------+|
      |              |
      +------|-------+
             |
        +----+----+
        |         |
        v         v
    +---------+ +-------------+
    | Primary | | Read Replica|
    |   DB    | |     DB      |
    +---------+ +-------------+

    App Servers
         |
         | photo files
         v
    +----------------+
    | Object Storage |
    +----------------+
         |
         | photo delivery
         v
        CDN

    App Servers
         |
         | thumbnail job
         v
    +-------------+
    |    Queue    |
    +-------------+
         |
         v
    +-------------+
    |   Worker    |
    +-------------+
         |
         | creates thumbnail
         v
    +----------------+
    | Object Storage |
    +----------------+

## Component Explanations

### Users / Clients

Users and clients send requests to upload photos and view their feeds.

### CDN

The CDN stores cached copies of static files, photos and thumbnails close to users so they load faster and reduce load on the main servers.

### Load Balancer

The load balancer spreads incoming API requests across healthy app servers so one server does not receive all the traffic.

### App Servers

The app servers handle application logic such as authentication, feed requests and photo upload requests.

### Cache

The cache stores frequently requested information such as feed results so the system can respond quickly without querying the database every time.

### Primary Database

The primary database stores important structured data and handles writes such as new photo records, users and follows.

### Read Replica

The read replica copies data from the primary database and handles many read requests so the primary database has less work.

### Object Storage

Object storage keeps the large original photo files and thumbnail files instead of storing them inside the database.

### Queue

The queue stores background jobs such as requests to create thumbnails so the user does not need to wait for the work to finish.

### Worker

The worker takes jobs from the queue, creates thumbnails and saves the finished thumbnails in object storage.

## Photo Upload Flow

1. The user chooses a photo and sends an upload request to SnapShare.

2. The request reaches the load balancer.

3. The load balancer sends the request to a healthy app server.

4. The app server checks the user's authentication and validates the upload.

5. The original photo is saved in object storage.

6. The app server creates a database record containing information about the photo, including the owner, upload time and object storage location.

7. The app server adds a thumbnail creation job to the message queue.

8. The app server can reply to the user without waiting for the thumbnail to be created.

9. A worker takes the thumbnail job from the queue.

10. The worker reads the original image from object storage.

11. The worker creates the 50 KB thumbnail.

12. The worker saves the thumbnail in object storage.

13. The database record can be updated with the thumbnail location if needed.

14. The CDN can cache the original photo and thumbnail when users request them.

15. When followers open their feeds, the app can use the cache and read replica to return feed information quickly while the CDN delivers the photo files.

## Trade-Offs

### Speed versus Freshness

Caching feed data makes requests much faster and reduces database load, but cached feeds may briefly contain old information. The system must invalidate or expire cached data when appropriate.

### Cost versus Reliability

Using multiple app servers, a CDN, a read replica, object storage and workers makes the system more reliable and scalable, but it also costs more than running everything on one server.

### Simplicity versus Scalability

A single server would be easier to build and manage, but it would struggle with SnapShare's traffic and would be a single point of failure. The scaled design is more complex but can handle much more traffic.

### Fast Upload Response versus Immediate Thumbnail Availability

Using a queue lets the upload request finish quickly because thumbnail creation happens in the background. The trade-off is that the thumbnail may not be available immediately after the user uploads the photo.