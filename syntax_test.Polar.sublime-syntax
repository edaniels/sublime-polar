// SYNTAX TEST "Packages/Polar/Polar.sublime-syntax"

actor User {
// <- source.polar meta.resource-block keyword.control
//    ^^^^ source.polar entity.name.type
}

# Superadmins can do everything across the entire application
// <- source.polar comment.line.number-sign
// ^ source.polar comment.line.number-sign

global {
// ^ keyword.control

  roles = ["GitHub staff"];
//        ^ source.polar meta.bracket.list

}


resource Organization {
// <- meta.resource-block keyword.control
//           ^^^^^^^^ meta.resource-block entity.name.type

  roles = ["owner", "member", "admin"];
//           ^^^^^ meta.resource-block meta.bracket.list string.quoted.double

  permissions = ["view", "rename", "delete", "invite", "update"];


  # Admins can do everything members can

  "member" if "owner";
//         ^^ meta.resource-block constant.character

  # member permissions

  "view" if "member";

  # owner permissions

  "rename" if "owner";

  "delete" if "owner";

  "invite" if "owner";

  "member" if "admin";

}


# Anyone can access public resources

# Resources are organized into a hierarchy, and granting access to a parent grants access to its children

resource Repository {

  roles = ["member", "admin"];

  permissions = ["view", "clone", "update", "delete"];

  relations = {

    # A Repository can have a parent Organization

    parent: Organization,
//            ^^^^^^^^^^ meta.resource-block meta.relation-declaration entity.name.type.resource

  };


  # anyone can access any public resource

  "view" if is_public(resource);

  "clone" if is_public(resource);

  # if a user has a role on a(n) Organization they also have that role on the Organization's Repository

  role if role on "parent";
//             ^^ meta.resource-block constant.character

  "view" if "member";

  "update" if "admin";

  "delete" if "admin";

}


resource ExampleOrganization {

  roles = ["admin"];

  permissions = ["rename", "delete"];


  "rename" if "admin";

  "delete" if "admin";

  "admin" if global "GitHub staff";

}


test "Users can be admins or members of their tenant" {

  setup {

    # Alice is an admin of the BigCo tenant

    has_role(User{"alice"}, "owner", Organization{"BigCo"});
//                                       ^ source.polar meta.test-block meta.test-setup constant.other.object-literal entity.name.type.resource

    # Mary is a member of the BigCo tenant

    has_role(User{"mary"}, "member", Organization{"BigCo"});

  }

  # Alice can perform administrative actions on the BigCo tenant

  assert allow(User{"alice"}, "view", Organization{"BigCo"});
// ^^^^^ meta.test-block keyword.other
//       ^^^^^ meta.test-block support.function.rule
//             ^^^^ meta.test-block constant.other.object-literal entity.name.type.resource
  # Mary cannot perform administrative actions on the BigCo tenant

  assert_not allow(User{"mary"}, "rename", Organization{"BigCo"});

  assert_not allow(User{"mary"}, "delete", Organization{"BigCo"});

  assert_not allow(User{"mary"}, "invite", Organization{"BigCo"});

  # Mary can perform member actions on the BigCo tenant

  assert allow(User{"mary"}, "view", Organization{"BigCo"});

  # Mary cannot perform member actions on a tenant she is not a member of

  assert_not allow(User{"mary"}, "view", Organization{"MegaCorp"});

}


test "Anyone can acdess public resources" {

  setup {

    is_public(Repository{"ABC"});

  }

  # Alice can access Repository ABC because it is public.

  assert allow(User{"alice"}, "view", Repository{"ABC"});

  assert allow(User{"alice"}, "clone", Repository{"ABC"});

  # Alice cannot access Repository DEF because it isn't public.

  assert_not allow(User{"alice"}, "view", Repository{"DEF"});

  assert_not allow(User{"alice"}, "clone", Repository{"DEF"});

}


test "Parent-child permissions" {

  setup {

    # Alice is an admin of the Organization

    has_role(User{"alice"}, "admin", Organization{"parent_a"});

    # This Repository belongs to this Organization

    has_relation(Repository{"child_a"}, "parent", Organization{"parent_a"});

  }

  # So Alice can also update the Repository

  assert allow(User{"alice"}, "update", Repository{"child_a"});

}


test "Global superadmins get the admin role on every ExampleOrganization" {

  setup {

    has_role(User{"alice"}, "GitHub staff");

  }

  assert allow(User{"alice"}, "rename", ExampleOrganization{"BigCo"});

  assert allow(User{"alice"}, "delete", ExampleOrganization{"MegaCorp"});

}
