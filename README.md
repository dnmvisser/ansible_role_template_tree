# ansible_role_template_tree

Deploy an entire directory of templates.

Benefits over template + filetree/fileglob lookup:

- Supports deletion of remote content that is not available locally (i.e.
  synching).
- Configurable set of file names extensions to template, so that other extension are
  not templated but copied. Useful for mixed content (scripts/images/blobs).
- Configurable validation per file name regex. Useful to validate scripts files,
  but also verify the integrity of images, etc.
