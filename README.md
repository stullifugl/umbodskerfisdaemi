Ég úbjó stutt sýnidæmi um hvernig ég held að einföld útgáfa af umboðskerfisgagnagrunninum geti verið settur upp.\
Gagnagrunnurinn geymir lýsingu á því hvað felst með hverju umboði fyrir sig og framenda væri hægt að smíða ofan á þjónustu sem talar við\
gagnagrunninn sem sér um umsýsluna á gögnunum.


Gögn frá get fyrirspurn sem að nær í gögn um ákveðið umboð fyrir ákveðið fyrirtæki eða einstakling, t.d. Fugla, gæti þá litið einhvern veginn svona út
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

Get query-a sem að nær í gögn fyrir client (eins og island.is) um ákveðinn notenda með kennitölu "111111-1111"
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
Hugmyndin hér er að display_name er gildið sem að client-arnir noti til að birta module-ana á island.is
####
####
####
####

OPA útfærslan gæti þá litið einhvern veginn svona út

Reglur og poliy-ur
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


Input frá rest endapunkti inn í OPA
```
{
    "user": "111111-1111",
    "delegation_actor": "Fuglar",
    "endpoint": "https://www.rsk.is/einstaklingar/arsreikningar/skattar_og_gjold/virdisaukaskattur",
	"controller": "virdisaukaskattur",
    "client_name": "rsk"
}
```

Gögn sem eru sótt frá umboðskerfisgagnagrunninum út frá input gildunum. 
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

Output-ið frá OPA til þjónustunnar verður þá
```
{
  "allow": true,
  "endpoint_is_allowed": true,
  "user_is_delegated": true
}
```
