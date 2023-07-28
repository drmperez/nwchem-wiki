## ZCOORD: Forcing internal coordinates

By default redundant internal coordinates are generated for use in
geometry optimizations. Connectivity is inferred by comparing
inter-atomic distances with the sum of the van der Waals radii of the
two atoms involved in a possible bond, times a scaling factor. The
scaling factor is an input parameter of ZCOORD which maybe changed from
its default value of 1.3. Under some circumstances (unusual bonding,
bond dissociation, ...) it will be necessary to augment the
automatically generated list of internal coordinates to force some
specific internal coordinates to be included in among the internal
coordinates. This is accomplished by including the optional directive
ZCOORD within the geometry directive. The general form of the ZCOORD
directive is as
follows:
```
 ZCOORD  
    CVR_SCALING <real value>  
    BOND    <integer i> <integer j> \  
            [<real value>] [<string name>] [constant]  
    ANGLE   <integer i> <integer j> <integer k> \  
            [<real value>] [<string name>] [constant]` 
    TORSION <integer i> <integer j> <integer k> <integer l> \  
            [<real value>] [<string name>] [constant]  
 END
```
The centers i, j, k and l must be specified using the numbers of the
centers, as supplied in the input for the Cartesian coordinates. The
`ZCOORD` input parameters are defined as follows:

  - `cvr_scaling` -- scaling factor applied to van der Waals radii.
  - `bond` -- a bond between the two centers.
  - `angle` -- an angle among the three atoms i, j and k.
  - `torsion` -- a torsion (or dihedral) angle. The angle between the
    planes i-j-k and j-k-l.

A value may be specified for a user-defined internal coordinate, in
which case it is forced upon the input Cartesian coordinates while
attempting to make only small changes in the other internal coordinates.
If no value is provided the value implicit in the input coordinates is
kept. If the keyword constant is specified, then that internal variable
is not modified during a geometry optimization with
[DRIVER](Geometry-Optimization.md#geometry-optimization-with-driver).
Each internal coordinate may also be named either for easy
identification in the output, or for the application of constraints
([Applying constraints in geometry
optimizations](#applying-constraints-in-geometry-optimizations)).

If the keyword adjust is specified on the main GEOMETRY directive, only
ZCOORD data may be specified and it can be used to change the
user-defined internal coordinates, including adding/removing constraints
and changing their values.

## Applying constraints in geometry optimizations

Internal coordinates specified as constant in a ZCOORD directive or in
the constants section of a ZMATRIX directive, will be frozen at their
initial values if a geometry optimization is performed with DRIVER
(Section 20).

If internal coordinates have the same name (give or take an optional
sign for torsions) then they are forced to have the same value. This may
be used to force bonds or angles to be equal even if they are not
related by symmetry.

When atoms have been specified by their Cartesian coordinates, and
internal coordinates are not being used, it is possible to freeze the
cartesian position of selected atoms. This is useful for such purposes
as optimizing a molecule absorbed on the surface of a cluster with fixed
geometry. Only the gradients associated with the active atoms are
computed. This can result in a big computational saving, since gradients
associated with frozen atoms are forced to zero (Note, however, that
this destroys the translational and rotational invariance of the
gradient. This is not yet fully accommodated by the
[STEPPER](Geometry-Optimization.md#geometry-optimization-with-stepper)
geometry optimization software, and can sometimes result in slower
convergence of the optimization. The DRIVER optimization package does
not suffer from this problem).

The [SET](SET.md) directive is used to freeze atoms,
by specifying a directive of the form:
```
 set geometry:actlist <integer list_of_center_numbers>
```
This defines only the centers in the list as active. All other centers
will have zero force assigned to them, and will remain frozen at their
starting coordinates during a geometry optimization.

For example, the following directive specifies that atoms numbered 1, 5,
6, 7, 8, and 15 are active and all other atoms are frozen:
```
 set geometry:actlist 1 5:8 15
```
or equivalently,
```
 set geometry:actlist 1 5 6 7 8 15
```
If this option is not specified by entering a [SET](SET.md)
directive, the default behavior in the code is to treat all atoms as
active. To revert to this default behavior after the option to define
frozen atoms has been invoked, the [UNSET](UNSET.md) directive
must be used. The form of the UNSET directive is as follows:
```
 unset geometry:actlist
```

## How to use the adjust keyword

The following example will impose a constraint on the K-O bond with a length of three angstroms.
```
geometry
 K 0.  0.0000  0.0000
 O 0.  0.0000  3.2000
 H 0. -0.7644  3.8162
 H 0.  0.7644  3.8162
end
geometry adjust
 zcoord
  bond  1 2   3. ko  constant
 end
end
```
