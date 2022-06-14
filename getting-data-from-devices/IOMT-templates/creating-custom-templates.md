# Creating Custom Templates

Consult the [IOMT-FHIR documentation](https://github.com/microsoft/iomt-fhir/blob/master/docs/Configuration.md) for information on writing custom templates.

You can also inspect the templates in the `mapping-templates` folder to see how the current ones look, and use the information in the [troubleshooting guide](troubleshooting.md) to validate your templates.

## Add new custom templates to the IOMT connector

1. Create a new GIT branch in the repository
1. Add any new device templates required to the `mapping-templates/devices` folder.
1. Add corresponding FHIR mapping templates to the `mapping-templates/fhir` folder.
1. Create a Pull Request to update the template store.
