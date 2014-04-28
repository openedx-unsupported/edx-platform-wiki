Currently a WIP

What you should do in your new glorious opaque keys future.

For the most part, extending the platform should not be substantially different. Now instead of passing around `course_id` strings and `location` strings, we will now be passing around OpaqueKey objects.

## Constructing Opaque Keys

In general, the best way to construct an opaque key is to use the correct constructor for the correct type of opaque key.

For example:
```
course_key = SlashSeparatedCourseKey('org', 'course', 'run')
location = Location('org', 'course', 'run', 'category', 'name', 'revision')
```

As of right now, the use of opaque keys (especially on the LMS) is mostly reserved for in-memory representations. While we are in the process of trying to update and migrate old data correctly, you might need to construct an opaque key out of an old-style string-representation of the `course_id` or `location`. This is especially true when it comes to urls or external services that send across the old-style strings. We have a few standard patterns for parsing the old-style strings correctly.

For constructing course keys:
```
SlashSeparatedCourseKey.from_deprecated_string('org/course/run')
```

For 



## Getting information out of Opaque Keys

It is possible to get information from these objects. For example, if you are given a `course_key`, you can use `course_key.org` to get the organization the course belongs to.