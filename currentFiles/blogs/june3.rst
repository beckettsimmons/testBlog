Tuesday June 3rd.
==================

Started off the day with installing as many packages as possible onto my python34  virtualenv. Doing this with just a simple *requirements.txt* file and *pip \-r* and slowly uncommenting out packages one at a time.

One issue is that the requirements file included some package in which pip couldn't find. So I used this command to list all of packages versions and then choose the most applicable one. ::

    pip install --no-deps --no-install <package> -v


