
---
sort: 20
---

# RDF syntax

class_template_mappings:
      "http://xmlns.com/foaf/0.1/Person": "person.html"
instance_template_mappings:
      "http://aksw.org/Team": "team.html"



restriction: "SELECT ?resourceUri WHERE { ?resourceUri <http://www.ifi.uio.no/INF3580/family#hasFather> <http://www.ifi.uio.no/INF3580/simpsons#Homer> }"

{% assign resource = '<http://www.ifi.uio.no/INF3580/simpsons#Homer>' | rdf_get %}
{{ resource | rdf_property: '<http://xmlns.com/foaf/0.1/job>' }}