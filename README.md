I made a short example of one version of how the registration database could be set up.\
The database stores a description of what each registration beholds and a front-end could be build upon a service that talks to the \
database that handles the registration data.


Data from a get http request that fetches data about a certain registration for a company or an individual, f.x. Fuglar ehf, could then look something like this
```
{
    "info": {
        "delegation_unique_name": "is_fjarmal_arsreikningar",
        "display_name": "@island.is/fjarmal/arsreikningar",
        "description": "Umboð sem veitir client-um aðgang að ársreikninga fjármálamodúl island.is",
        "delegation_creator": "island.is"
    },
    "natural_delegation_settings": {
        "allow_all_natural_registries": false,
        "allow_parental_registries": false,
        "allow_company_owner_registries": true
    },
    "client_delegations": [
        {
            "client_name": "island.is_vefsida",
            "creator": "island.is"
        },
        {
            "client_name": "island.is_app",
            "creator": "island.is"
        }
    ],
    "delegation_end_points": [
        {
            "rest_end_point": "https://www.rsk.is/einstaklingar/arsreikningar/skattar_og_gjold/virdisaukaskattur",
            "creator": "RSK"
        },
        {
            "rest_end_point": "https://www.rsk.is/einstaklingar/skattar-og-gjold/tekjuskattur-og-utsvar/",
            "creator": "RSK"
        },
        {
            "rest_end_point": "https://www.lsr.is/avinnsla/a-deild/",
            "creator": "LSR"
        }
    ],
    "client_controller_delegations": [
        {
            "controller_name": "innheimta",
            "client_name": "rsk",
            "creator": "RSK"
        }
    ],
    "delegation_groups": [
        {
            "delegation_actor": "Fuglar",
            "creator": "Fuglar",
            "user_group": [
                {
                    "name": "Ársreikningar fugla hópurinn",
                    "description": "Notendahópur sem hefur aðgang að því að sjá alla ársreikninga Fugla",
                    "creator": "Fuglar",
                    "users": [
                        {
                            "national_id": "111111-1111",
                            "name": "Sigurður Sturla Bjarnason",
                            "email": "blabla@fuglar.com"
                        },
                        {
                            "national_id": "222222-2222",
                            "name": "Unnar Bjarnason",
                            "email": "blabla@fuglar.com"
                        },
                        {
                            "national_id": "333333-3333",
                            "name": "Valur Einarsson",
                            "email": "blabla@fuglar.com"
                        }
                    ]
                }
            ]
        }
    ]
}
```

A get http reuest that fetches data for a client (like island.is) about a certain user with a social security number "111111-1111"
```
{
    "user": {
        "national_id": "111111-1111",
        "name": "Sigurður Sturla Bjarnason",
        "email": "blabla@gmail.com"
    },
    "delegations": [
        {
            "info": {
                "delegation_unique_name": "is_fjarmal_arsreikningar",
                "display_name": "@island.is/fjarmal/arsreikningar",
                "description": "Umboð sem stýrir hvaða starfsmenn fyrirtækis geta fengið aðgang að ársreikninga fjármálamodúl island.is",
                "delegation_creator": "island.is"
            },
            "delegation_actor": {
                // info um Fugla hér
            }
        },
        {
            "info": {
                "delegation_unique_name": "inna_einkunnir",
                "display_name": "@inna_birta_einkunnir",
                "description": "Umboð sem veitir einstakling leyfi til að gefa öðrum einstaklingum aðgang að sjá einkunnir sínar í einkunnamódul island.is",
                "delegation_creator": "island.is"
            },
            "delegation_actor": {
                // info um mitt eina ímyndaða barn mitt hér
            }
        }
    ]
}
```
The idea here is that the display_name is the value that the clients use to publish the modules on the island.is front-end
####
####
####
####

OPA implementation then could look something like this

Rules and policies
```
package app.abac

default allow = false

allow {
 	user_is_delegated
  endpoint_is_allowed
}

allow {
	user_is_delegated
  controller_is_allowed
}

endpoint_is_allowed {
	some i
  some j
  
  data.delegations[i].delegation_end_points[j].rest_end_point = input.endpoint
}

controller_is_allowed {
	some i
  some j
  
  data.delegations[i].client_controller_delegations[j].controller_name == input.controller
  data.delegations[i].client_controller_delegations[j].client_name == input.client_name
}

user_is_delegated {
  some i
  some j
  some x
  some z

  data.delegations[i].delegation_groups[j].user_group[x].users[z].national_id == input.user
}
```


Input from a rest endpoint into OPA
```
{
    "user": "111111-1111",
    "delegation_actor": "Fuglar",
    "endpoint": "https://www.rsk.is/einstaklingar/arsreikningar/skattar_og_gjold/virdisaukaskattur",
	"controller": "virdisaukaskattur",
    "client_name": "rsk"
}
```

Data collected from the registration database from the input values
```
{
  "delegations": [
    {
      "info": {
        "delegation_unique_name": "is_fjarmal_arsreikningar",
        "display_name": "@island.is/fjarmal/arsreikningar",
        "description": "Umboð sem veitir client-um aðgang að ársreikninga fjármálamodúl island.is",
        "delegation_creator": "island.is"
      },
      "natural_delegation_settings": {
        "allow_all_natural_registries": false,
        "allow_parental_registries": false,
        "allow_company_owner_registries": true
      },
      "client_delegations": [
        {
          "client_name": "island.is_vefsida",
          "creator": "island.is"
        },
        {
          "client_name": "island.is_app",
          "creator": "island.is"
        }
      ],
      "delegation_end_points": [
        {
          "rest_end_point": "https://www.rsk.is/einstaklingar/arsreikningar/skattar_og_gjold/virdisaukaskattur",
          "creator": "RSK"
        },
        {
          "rest_end_point": "https://www.rsk.is/einstaklingar/skattar-og-gjold/tekjuskattur-og-utsvar/",
          "creator": "RSK"
        },
        {
          "rest_end_point": "https://www.lsr.is/avinnsla/a-deild/",
          "creator": "LSR"
        }
      ],
      "client_controller_delegations": [
        {
          "controller_name": "innheimta",
          "client_name": "rsk",
          "creator": "RSK"
        }
      ],
      "delegation_groups": [
        {
          "delegation_actor": "Fuglar",
          "creator": "Fuglar",
          "user_group": [
            {
              "name": "Ársreikningar fugla hópurinn",
              "description": "Notendahópur sem hefur aðgang að því að sjá alla ársreikninga Fugla",
              "creator": "Fuglar",
              "users": [
                {
                  "national_id": "111111-1111",
                  "name": "Sigurður Sturla Bjarnason",
                  "email": "blabla@fuglar.com"
                },
                {
                  "national_id": "222222-2222",
                  "name": "Unnar Bjarnason",
                  "email": "blabla@fuglar.com"
                },
                {
                  "national_id": "333333-3333",
                  "name": "Valur Einarsson",
                  "email": "blabla@fuglar.com"
                }
              ]
            }
          ]
        }
      ]
    }
  ]
}
```

The output from OPA to the REST service then will be
```
{
  "allow": true,
  "endpoint_is_allowed": true,
  "user_is_delegated": true
}
```
