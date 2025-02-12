Euclidean neural networks
=========================

What is ``e3nn``?
-----------------

``e3nn`` is a python library based on pytorch_ to create equivariant neural networks for the group :math:`O(3)`.

Where to start?
---------------

- Guide to the `e3nn.o3.Irreps`: :ref:`irreps guide`
- Guide to implement a :ref:`conv guide`
- The simplest example to start with is :ref:`tetris_poly`.
- Guide to implement a :ref:`transformer guide`

.. toctree::
    :maxdepth: 2

    api/e3nn
    guide/guide
    examples/examples

Demonstration
-------------

All the functions to manipulate rotations (rotation matrices, Euler angles, quaternions, convertions, ...) can be found here :ref:`Rotation functions`.
The irreducible representations of :math:`O(3)` (more info at :ref:`Irreducible representations`) are represented by the class `e3nn.o3.Irrep`.
The direct sum of multiple irrep is described by an object `e3nn.o3.Irreps`.

If two tensors :math:`x` and :math:`y` transforms as :math:`D_x = 2 \times 1_o` (two vectors) and :math:`D_y = 0_e + 1_e` (a scalar and a pseudovector) respectively, where the indices :math:`e` and :math:`o` stand for even and odd -- the representation of parity,

.. jupyter-execute::

    import torch
    from e3nn import o3

    irreps_x = o3.Irreps('2x1o')
    irreps_y = o3.Irreps('0e + 1e')

    x = irreps_x.randn(-1)
    y = irreps_y.randn(-1)

    irreps_x.dim, irreps_y.dim


their outer product is a :math:`6 \times 4` matrix of two indices :math:`A_{ij} = x_i y_j`.

.. jupyter-execute::

    A = torch.einsum('i,j', x, y)
    A


If a rotation is applied to the system, this matrix will transform with the representation :math:`D_x \otimes D_y` (the tensor product representation).

.. math::

    A = x y^t \longrightarrow A' = D_x A D_y^t

Which can be represented by

.. jupyter-execute::
    :hide-code:

    import matplotlib.pyplot as plt

.. jupyter-execute::

    R = o3.rand_matrix()
    D_x = irreps_x.D_from_matrix(R)
    D_y = irreps_y.D_from_matrix(R)

    plt.imshow(torch.kron(D_x, D_y), cmap='bwr', vmin=-1, vmax=1);


This representation is not irreducible (is reducible) (needs explaination). It can be decomposed into irreps by a change of basis. The outerproduct followed by the change of basis is done by the class `e3nn.o3.FullTensorProduct`.

.. jupyter-execute::

    tp = o3.FullTensorProduct(irreps_x, irreps_y)
    print(tp)

    tp(x, y)


As a sanity check, we can verify that the representation of the tensor product is block diagonal and of the same dimension.

.. jupyter-execute::

    D = tp.irreps_out.D_from_matrix(R)
    plt.imshow(D, cmap='bwr', vmin=-1, vmax=1);


`e3nn.o3.FullTensorProduct` is a special case of `e3nn.o3.TensorProduct`. Another example is `e3nn.o3.FullyConnectedTensorProduct`, which contains learnable weights, making it useful to create neural networks.


.. _pytorch: https://pytorch.org/

