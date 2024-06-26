# Description
Export a PySB model to JSON using the JsonExporter class. This example demonstrates how to generate a JSON representation of a PySB model.

# Code
```
from pysb.export import Exporter
from pysb.bng import generate_equations
import json
from pysb.core import Model, MultiState, KeywordMeta, Parameter, Expression

class PySBJSONEncoder(json.JSONEncoder):
    PROTOCOL = 2
    @classmethod
    def encode_keyword(cls, keyword):
        return {'__object__': str(keyword)}
    @classmethod
    def encode_multistate(cls, stateval):
        return {
            '__object__': '__multistate__',
            'sites': stateval.sites
        }
    @classmethod
    def encode_monomer(cls, mon):
        return {
            'name': mon.name,
            'sites': mon.sites,
            'states': mon.site_states
        }
    @classmethod
    def encode_compartment(cls, cpt):
        return {
            'name': cpt.name,
            'parent': cpt.parent.name if cpt.parent else None,
            'dimension': cpt.dimension,
            'size': cpt.size.name if cpt.size else None
        }
    @classmethod
    def encode_parameter(cls, par):
        return {
            'name': par.name,
            'value': par.value
        }
    @classmethod
    def encode_expression(cls, expr):
        return {
            'name': expr.name,
            'expr': expr.expr.name if isinstance(
                expr.expr, (Parameter, Expression)) else str(expr.expr)
        }
    @classmethod
    def encode_monomer_pattern(cls, mp):
        return {
            'monomer': mp.monomer.name,
            'site_conditions': mp.site_conditions,
            'compartment': mp.compartment.name if mp.compartment else None,
            'tag': mp._tag.name if mp._tag else None
        }
    @classmethod
    def encode_complex_pattern(cls, cp):
        return {
            'monomer_patterns': [cls.encode_monomer_pattern(mp)
                                 for mp in cp.monomer_patterns],
            'compartment': cp.name if cp.compartment else None,
            'match_once': cp.match_once,
            'tag': cp._tag.name if cp._tag else None
        }
    @classmethod
    def encode_reaction_pattern(cls, rp):
        return {
            'complex_patterns': [cls.encode_complex_pattern(cp)
                                 for cp in rp.complex_patterns]
        }
    @classmethod
    def encode_rule_expression(cls, rexp):
        return {
            'reactant_pattern': cls.encode_reaction_pattern(
                rexp.reactant_pattern),
            'product_pattern': cls.encode_reaction_pattern(
                rexp.product_pattern),
            'reversible': rexp.is_reversible
        }
    @classmethod
    def encode_rule(cls, r):
        return {
            'name': r.name,
            'rule_expression': cls.encode_rule_expression(r.rule_expression),
            'rate_forward': r.rate_forward.name,
            'rate_reverse': r.rate_reverse.name if r.rate_reverse else None,
            'delete_molecules': r.delete_molecules,
            'move_connected': r.move_connected,
            'energy': r.energy,
        }
    @classmethod
    def encode_energypattern(cls, ep):
        return {
            'name': ep.name,
            'pattern': cls.encode_complex_pattern(ep.pattern),
            'energy': ep.energy.name,
        }
    @classmethod
    def encode_observable(cls, obs):
        return {
            'name': obs.name,
            'reaction_pattern': cls.encode_reaction_pattern(
                obs.reaction_pattern),
            'match': obs.match
        }
    @classmethod
    def encode_initial(cls, init):
        return {
            'pattern': cls.encode_complex_pattern(init.pattern),
            'parameter_or_expression': init.value.name,
            'fixed': init.fixed
        }
    @classmethod
    def encode_annotation(cls, ann):
        return {
            'subject': 'model' if isinstance(ann.subject, Model)
            else ann.subject.name,
            'object': str(ann.object),
            'predicate': str(ann.predicate)
        }
    @classmethod
    def encode_tag(cls, tag):
        return {
            'name': tag.name
        }
    @classmethod
    def encode_model(cls, model):
        d = dict(protocol=cls.PROTOCOL, name=model.name)
        encoders = {
            'monomers': cls.encode_monomer,
            'compartments': cls.encode_compartment,
            'tags': cls.encode_tag,
            'parameters': cls.encode_parameter,
            'expressions': cls.encode_expression,
            'rules': cls.encode_rule,
            'energypatterns': cls.encode_energypattern,
            'observables': cls.encode_observable,
            'initials': cls.encode_initial,
            'annotations': cls.encode_annotation
        }
        for component_type, encoder in encoders.items():
            d[component_type] = [encoder(component)
                                 for component in
                                 getattr(model, component_type)]
        return d
    def default(self, o):
        if isinstance(o, Model):
            return self.encode_model(o)
        elif isinstance(o, MultiState):
            return self.encode_multistate(o)
        elif isinstance(o, KeywordMeta):
            return self.encode_keyword(o)
        return super(PySBJSONEncoder, self).default(o)

class PySBJSONWithNetworkEncoder(PySBJSONEncoder):
    @classmethod
    def encode_reaction(cls, rxn):
        rxn = rxn.copy()
        rxn['rate'] = rxn['rate'].name if isinstance(rxn['rate'], (Parameter, Expression)) else str(rxn['rate'])
        return rxn
    @classmethod
    def encode_observable(cls, obs):
        o = super(PySBJSONWithNetworkEncoder, cls).encode_observable(obs)
        o['species'] = obs.species
        o['coefficients'] = obs.coefficients
        return o
    @classmethod
    def encode_model(cls, model):
        d = super(PySBJSONWithNetworkEncoder, cls).encode_model(model)
        generate_equations(model)
        additional_encoders = {
            '_derived_parameters': cls.encode_parameter,
            '_derived_expressions': cls.encode_expression,
            'reactions': cls.encode_reaction,
            'reactions_bidirectional': cls.encode_reaction,
            'species': cls.encode_complex_pattern
        }
        for component_type, encoder in additional_encoders.items():
            d[component_type] = [encoder(component) for component in getattr(model, component_type)]

    def export(self, include_netgen=False):
        """Generate the corresponding JSON for the PySB model associated
        with the exporter.

        Parameters
        ----------
        include_netgen: bool
            Include cached network generation data (reactions, species,
            local function-derived parameters and expressions) if True.

        Returns
        -------
        string
            The JSON output for the model.
        """
        return json.dumps(self.model, cls=PySBJSONWithNetworkEncoder

```
