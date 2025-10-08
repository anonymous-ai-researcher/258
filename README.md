## Environment Requirement

1. jdk 1.8
2. IDE: IDEA
3. All dependencies are in the folder `Dependency`

## Run configuration

1. Unzip the entire project.
2. Import it into IDEA.
3. Make the src folder as source root.
4. Make sure that the jar package under the `Dependency` folder has been added into project as libriaries.

### Data

Datasets **Oxford-ISG** and **BioPortal**  are in compressed package **test_forgetting.zip**. 

## Compute Role Forgetting


The code can be used to compute the solution of strong role forgetting for $\mathcal{ALCQ}$ ontologies. There is a code template showing how to compute the forgetting result.

```java
package evaluation;

import concepts.AtomicConcept;
import convertion.BackConverter;
import convertion.Converter;
import forgetting.Fame;
import formula.Formula;
import org.semanticweb.owlapi.apibinding.OWLManager;
import org.semanticweb.owlapi.io.IRIDocumentSource;
import org.semanticweb.owlapi.io.OWLXMLOntologyFormat;
import org.semanticweb.owlapi.model.*;
import roles.AtomicRole;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.OutputStream;
import java.util.List;
import java.util.Set;

public class demo {
  public void main (String[] args) throws OWLOntologyCreationException, CloneNotSupportedException, FileNotFoundException, OWLOntologyStorageException {
    OWLOntologyManager manager = OWLManager.createOWLOntologyManager();

    /* TODO: Input your target ontology path */
    String filePath = "";

    /* TODO: Enter the save path of the UI. */
    String OutPutPath = "";


    File file = new File(filePath);
    IRI iri = IRI.create(file);
    OWLOntology ontology = manager.loadOntologyFromOntologyDocument(new IRIDocumentSource(iri),
        new OWLOntologyLoaderConfiguration().setLoadAnnotationAxioms(true));
    Converter converter = new Converter();
    converter.CReset();
    Fame fame = new Fame();
    List<AtomicRole> roleList = converter.getRolesInSignature_ShortForm(ontology);

    /* roleList contains all role names that occur in the input ontology.
    TODO：Write your code to select the names to be eliminated from the input ontology.
    Maybe you can implement a select function.
    Set<AtomicRole> r_sig = select(roleList);
     */
    Set<AtomicRole> r_sig = null;
    OWLOntology ontoSol = null;

    BackConverter backConverter = new BackConverter();
    List<Formula> formula_list = converter.OntologyConverter_ShortForm(ontology);
    List<Formula> result_list = fame.FameCR(r_sig, formula_list);
    ontoSol = backConverter.toOWLOntology(result_list);
    File outFile = new File(OutPutPath);
    OutputStream os = new FileOutputStream(outFile);
    manager.saveOntology(ontoSol, new OWLXMLOntologyFormat(), os);

  }
}
```

