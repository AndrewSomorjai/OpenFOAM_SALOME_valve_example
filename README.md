# OpenFOAM_SALOME_valve_example
An example of using a Salome meshed unv file with OpenFOAM. 
This simuation simple runs an example of a valve in a pipe,
something similar to a butterfly valve. The inlet and outlet
of the pipes have no flow velocity and this just runs
a very simple case. 

This was tested with an AMD64 processor (32 gigs RAM)
on Xubuntu 18.04 LTS with OpenFOAM v18.06.
Make sure you have at least a terabyte of disk space to run
this because the results are 200 MB per timestep per processor.

Open meshing.hdf (after extracting meshing.hdf.7z.001 with 7zip) 
in Salome.Export stator mesh as stator.unv into fine_valve/stator
Export rotor mesh as rotor.unv into fine_valve/rotor

start command line in fine_valve/

follow the steps

	$ cd rotor 
	/rotor$ ideasUnvToFoam rotor.unv
	$ cd ../stator 
	/stator$ ideasUnvToFoam stator.unv
	/stator$ cd ../ 
	$ mergeMeshes -overwrite stator rotor
	$ cd rotor 
	/rotor$ checkMesh
	$ cd ../stator 
	/stator$ checkMesh
	/stator$ setSet
	Command>cellZoneSet rotor new setToCellZone region1
	Command>quit
	/stator$ topoSet

In your stator/constant/polyMesh/boundary file make sure AMI patches look like

	AMI2
    {
        type            cyclicAMI;//This needs to be changed.
		neighbourPatch  AMI1;//This needs to be added.
        nFaces          9774;
        startFace       4088405;
    }
    AMI1
    {
        type            cyclicAMI;//This needs to be changed.
		neighbourPatch  AMI2;//This needs to be added.
        nFaces          10160;
        startFace       4098179;
    }

	/stator$ decomposePar
	/stator$ mpiexec -np 2 renumberMesh -overwrite -parallel

In your parallel processor files ( e.g. processor1/constant/polyMesh/boundary)
the polyMesh boundary files need to look like this for patches wall and valve.

wall
    {
        type            wall;//this will cause an error if it's not wall
        nFaces          0;
        startFace       555107;
    }


 valve
    {
        type            wall;//this will cause an error if it's not wall
        nFaces          15752;
        startFace       566002;
    }
	
	
	/stator$ mpiexec -np 2 pimpleFoam -parallel | tee log .pimpleFoam

The simulation should now run....
