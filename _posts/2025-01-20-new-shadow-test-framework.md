---
layout:     post
title:      "New shadow test framework"
date:       2025-01-20 10:00:00 +0100
categories: test
background: '/assets/figures/2025-01-20-new-shadow-test-framework.jpg'
---

#  Victory! Shadow gets a new system test framework

I finally did it! It took a while, but shadow<sup>[1](#references)</sup> finally has a new system test framework. Once I got going, it didn't actually take that long, but it was hard to get started. Between one thing and another, I always ended up putting it off, but persistence pays off! But let's not get ahead of ourselves and start at the beginning.

# Background

Shadow is a very old project, and at least since it has had version control, it has always had a test framework. This was written in bash and allowed tests to be written in the same language. The framework was quite limited since one case could affect another. Also, it could only be run on Debian, since the return to the initial state was done based on the shadow files for this distribution. There are still some more problems, but these were the most limiting in my opinion. If you want to know more, you can visit the ticket<sup>[2](#references)</sup> that I opened proposing the new framework.

Don't get me wrong, the framework is fine considering the time it was written. But like everything in life, it can always be improved.

# Change

That's why in 2023 I set out to create something better. And it was at the end of that same year that I opened the ticket mentioned above. After some back and forth, the community agreed that a proof of concept should be created and reviewed before formally accepting the framework.

# POC

At the end of 2024, I rolled up my sleeves and got down to business. The new framework is based on pytest-mh<sup>[3](#references)</sup> and is written in Python. Object-oriented programming is very powerful and allows us to create a good software design while at the same time being simple, creating a framework that is both flexible and powerful. The problems of the previous framework have been eliminated, and now one test does not affect the next when it fails. Also, the tests can be run on several distributions; Debian, Fedora, and Alpine for sure because they are part of the CI, but surely more can be added easily. If you want to know more functionalities, I invite you to visit the PR<sup>[4](#references)</sup>.

# Tests

The tests are also written in Python and are very easy to write. If you don't believe me, look at the following example:

```python
@pytest.mark.topology(KnownTopology.Shadow)
def test_useradd__add_user(shadow: Shadow):
    shadow.useradd("tuser")

    result = shadow.tools.id("tuser")
    assert result is not None, "User should be found"
    assert result.user.name == "tuser", "Incorrect username"
    assert result.user.id == 1000, "Incorrect UID"

    result = shadow.tools.getent.group("tuser")
    assert result is not None, "Group should be found"
    assert result.name == "tuser", "Incorrect groupname"
    assert result.gid == 1000, "Incorrect GID"

    assert shadow.fs.exists("/home/tuser"), "Home folder should be found"
```

Explanation:

1. Add a user named `tuser`.
2. Check that the user has been created and contains the correct attributes.
3. Check that the group has been created and contains the correct attributes.
4. Check that the home folder has been created.

See how easy it is! Well, from now on it's like this with everything, and also the expectations are very clear and the error messages are understandable without having to look for any context.

# Conclusion

I still need to add the bindings for some of shadow's functionalities and document everything, but the first step, and the hardest, has already been taken, so from now on everything will be easier.

It has been a hard work full of obstacles, many times due to other priorities, but I am happy with the path taken. I hope to continue like this and have good news in the coming months with even better news.

# References

1. [<u>shadow repository</u>](https://github.com/shadow-maint/shadow/)
2. [<u>New system test framework proposal</u>](https://github.com/shadow-maint/shadow/issues/835)
3. [<u>pytest-mh</u>](https://pytest-mh.readthedocs.io/en/latest/)
4. [<u>System test framework PR</u>](https://github.com/shadow-maint/shadow/pull/1131)

