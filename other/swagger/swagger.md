``` yaml
swagger: "2.0"
info:
  title: Sample API
  description: API description in Markdown.
  version: 1.0.0

host: api.example.com
basePath: /v1
schemes:
  - https

paths:
  /users:
    get:
      summary: Returns a list of users.
      description: Optional extended description in Markdown.
      produces:
        - application/json
      responses:
        200:
          description: OK
```

consumes:
  - application/json
  - application/xml
  - application/x-www-form-urlencoded
produces:
  - application/json
  - application/xml
  - application/x-www-form-urlencoded

in: formData
in: query
in: body
type: array, boolean, integer, number, object, string

*   Request
GET
``` yaml
      parameters:
        - in: query
          name: user_id
          required: true
          type: integer
          minimum: 1
          description: The ID of the user to return.
        - in: query
          name: city_id
          required: true
          type: integer
          minimum: 1
          description: The ID of the user to return.
```

FORM
``` yaml
      parameters:
        - in: formData
          name: user_id
          required: true
          type: integer
          minimum: 1
          description: The ID of the user to return.
        - in: formData
          name: city_id
          required: true
          type: integer
          minimum: 1
          description: The ID of the user to return.
```

JSON
``` yaml
      parameters:
        - in: "body"
          name: "body"
          description: 成功
          schema:
            type: object
            description: ""
            properties:
              data:
                type: object
                description: ""
                properties:
                  time:
                    type: string
                    description: 当前服务器时间
                    example: "20:30:10"
              errmsg:
                type: string
                description: 错误信息，当errno非0的时候有内容，可直接在页面显示
                example: ""
              errno:
                type: integer
                description: 请求返回码，0才表示逻辑成功
                example: 0
              traceid:
                type: string
                description: 链路ID，追查错误使用
                example: 64454c135d25c58d58fd81d4267cf502
```

``` yaml

```
*   Response
``` yaml
      responses:
        200:
          description: 成功
          schema:
            type: object
            description: ""
            properties:
              data:
                type: object
                description: ""
                properties:
                  time:
                    type: string
                    description: 当前服务器时间
                    example: "20:30:10"
              errmsg:
                type: string
                description: 错误信息，当errno非0的时候有内容，可直接在页面显示
                example: ""
              errno:
                type: integer
                description: 请求返回码，0才表示逻辑成功
                example: 0
              traceid:
                type: string
                description: 链路ID，追查错误使用
                example: 64454c135d25c58d58fd81d4267cf502
        400:
          description: The specified user ID is invalid (e.g. not a number).
        404:
          description: A user with the specified ID was not found.
        default:
          description: Unexpected error
```