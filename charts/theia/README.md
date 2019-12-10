# Theia

# Persitent Volumes
The gcePersistentDisk "theia-workspace" must exist before this chart can be applied.  
The PersistentVolume manifest (pv.yaml) should only be applied ONCE or it should  be stored in another respotiory to prevent deletion of the PersistentVolume when the whole app is deleted.
  
Possible solutions to apply PersistentVolume only once:
- Introduce existingVolume in values.yaml and wrap contents of pv.yaml in an if-clause so that the manifest is only applied when existingVolume is not set.
- Create the PersistentVolume outside of this repository

Caution:  
Deleting the whole app WILL DELETE the PersistentVolume when it is stored in the same chart as the deployment.

## Recovery

When you delete a PVC, corresponding PV becomes Released. This PV can contain sensitive data (say credit card numbers) and therefore nobody can ever bind to it, 
even if it is a PVC with the same name and in the same namespace as the previous one - who knows who's trying to steal the data!

Admin intervention is required here. He has two options:
* Make the PV available to everybody - delete PV.Spec.ClaimRef, Such PV can bound to any PVC (assuming that capacity, access mode and selectors match)
* Make the PV available to a specific PVC - pre-fill PV.Spec.ClaimRef with a pointer to a PVC. Leave the PV.Spec.ClaimRef,UID empty, as the PVC does not to need exist at this point and you don't know PVC's UID. This PV can be bound only to the specified PVC.
(from https://github.com/kubernetes/kubernetes/issues/48609#issuecomment-314066616)

### Rebind

    $ kubectl patch pv my-pv-name -p '{"spec":{"claimRef": null}}'

