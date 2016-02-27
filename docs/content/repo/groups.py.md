# groups.py

This file lets you specify or dynamically build groups of [nodes](nodes.py.md) in your environment.

As with `nodes.py`, you define your groups as a dictionary:

	groups = {
	    'all': {
	        'member_patterns': (
	            r".*",
	        ),
	    },
	    'group1': {
	        'members': (
	            'node1',
	        ),
	    },
	}

All group attributes are optional.

<br>

# Group attribute reference

This section is a reference for all possible attributes you can define for a group:

	groups = {
	     'group1': {
	         # THIS PART IS EXPLAINED HERE
	         'bundles': ["bundle1", "bundle2"],
	         'member_patterns': [r"^cluster1\."],
	         'metadata': {'foo': "bar"},
	         'os': 'linux',
	         'subgroups': ["group2", "group3"],
	     },
	}

<br>

## bundles

A list of [bundle names](bundles.md) to be assigned to each node in this group.

<br>

## member_patterns

A list of regular expressions. Node names matching these expressions will be added to the group members.

Matches are determined using [the search() method](http://docs.python.org/2/library/re.html#re.RegexObject.search).

<br>

## members

A tuple or list of node names that belong to this group.

<br>

## metadata

A dictionary of arbitrary data that will be accessible from each node's `node.metadata`. For each node, BundleWrap will merge the metadata of all of the node's groups first, then merge in the metadata from the node itself.

Metadata is merged recursively by default, meaning nested dicts will overlay each other. Lists will be appended to each other, but not recursed into. In come cases, you want to overwrite instead of merge a piece of metadata. This is accomplished through the use of `bundlewrap.metadata.atomic()` and best illustrated as an example:

	from bundlewrap.metadata import atomic

	groups = {
	    'all': {
	        'metadata': {
	            'interfaces': {
	                'eth0': {},
	            },
	            'nameservers': ["8.8.8.8", "8.8.4.4"],
	            'ntp_servers': ["pool.ntp.org"],
	        },
	    },
	    'internal': {
	        'metadata':
	            'interfaces': {
	                'eth1': {},
	            },
	            'nameservers': atomic(["10.0.0.1", "10.0.0.2"]),
	            'ntp_servers': ["10.0.0.1", "10.0.0.2"],
	        },
	    },
	}

A node in both groups will end up with `eth0` *and* `eth1`.

The nameservers however are overwritten, so that nodes what are in both the "all" *and* the "internal" group will only have the `10.0.0.x` ones while nodes just in the "all" group will have the `8.8.x.x` nameservers.

The NTP servers are appended: a node in both groups will have all three nameservers.

<div class="alert alert-warning">BundleWrap will consider group hierarchy when merging metadata. For example, it is possible to define a default nameserver for the "eu" group and then override it for the "eu.frankfurt" subgroup. The catch is that this only works for groups that are connected through a subgroup hierarchy. Independent groups will have their metadata merged in an undefined order.</div>

<br>

## os

Changes the default OS (see [nodes.py](nodes.py.md)) for nodes in this group. Subgroups can override this for their parent groups.

<br>

## subgroups

A tuple or list of group names whose members should be recursively included in this group.