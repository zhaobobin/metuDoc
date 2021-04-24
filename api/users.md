## Users 用户接口



#### Schema





#### 用户列表

- `GET /api/v1/users`

- 响应状态码：200

- 响应体：

  ```js
  [
    {
      "id": "57ea257b3670ca3f44c5beb6",
      "author_id": "541bf9b9ad60405c1f151a03",
      "tab": "share",
      "content": "content",
      "last_reply_at": "2017-01-11T13:32:25.089Z",
      "good": false,
      "top": true,
      "reply_count": 155,
      "visit_count": 28176,
      "create_at": "2016-09-27T07:53:31.872Z",
    },
    {
      "id": "57ea257b3670ca3f44c5beb6",
      "author_id": "541bf9b9ad60405c1f151a03",
      "tab": "share",
      "content": "content",
      "title": "《一起学 Node.js》彻底重写完毕",
      "last_reply_at": "2017-01-11T10:20:56.496Z",
      "good": false,
      "top": true,
      "reply_count": 193,
      "visit_count": 47633,
    },
  ]
  ```



#### 用户详情

- `GET /api/v1/users/:id`

- 响应状态码：200

- 响应体：

  ```js
  {
      "id": "57ea257b3670ca3f44c5beb6",
      "author_id": "541bf9b9ad60405c1f151a03",
      "tab": "share",
      "content": "content",
      "last_reply_at": "2017-01-11T13:32:25.089Z",
      "good": false,
      "top": true,
      "reply_count": 155,
      "visit_count": 28176,
      "create_at": "2016-09-27T07:53:31.872Z",
    }
  ```

  