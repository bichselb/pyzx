Getting Started
===============

.. _gettingstarted:

PyZX can be installed as a package using pip::

	pip install pyzx

If you wish to use the demo notebooks or benchmark circuits, then the repository can be cloned from `Github <https://github.com/Quantomatic/pyzx>`_.

The best way to get started if you have cloned the repository is to run the `Getting Started notebook <https://github.com/Quantomatic/pyzx/blob/master/demos/gettingstarted.ipynb>`_ in Jupyter. This page contains the same general information as that notebook.

.. warning::
	The newer JupyterLab as opposed to the older Jupyter Notebook uses a different framework for widgets which is currently not compatible with the widgets used in PyZX. It is therefore recommended that you use the classic notebook interface. If you are using JupyterLab you can find this interface by going to 'Help -> Launch Classic Notebook'.

Let's start by importing the library::
	
	>>> import pyzx as zx

For all the examples in this documentation we will assume you have imported PyZX in this manner.

Quantum circuits in PyZX are represented by the :class:`~pyzx.circuit.Circuit` class. File in the supported formats (QASM, QC, Quipper) can easily be imported into PyZX::

	circuit = zx.Circuit.load("path/to/circuit.extension")

PyZX tries to automatically figure out in which format the circuit is represented. The :mod:`~pyzx.generate` module supplies several ways to generate random circuits::
	
	>>> circuit = zx.generate.CNOT_HAD_PHASE_circuit(qubits=4,depth=20,clifford=True)

If you are running inside a Jupyter notebook, circuits can be easily visualized::
	
	>>> zx.draw(circuit)

.. figure::  _static/clifford.png
   :align:   center

The default drawing method is to use the D3 Javascript library. When not running in a Jupyter notebook ``zx.draw`` returns a matplotlib figure instead. 
In PyZX, Hadamard gates are stored as edges between vertices, which here are shown as blue lines.

There are two main data structures in PyZX, Circuits and Graphs. A :class:`~pyzx.circuit.Circuit` is essentially just a list of gates. The above is an example of a ``Circuit``::

	>>>print(circuit.gates)
	[CNOT(2,3), CNOT(0,1), CNOT(1,3), CNOT(1,2), CNOT(1,3), S(0), S(1), S(3), HAD(2), HAD(3), CNOT(2,1), HAD(3), CNOT(2,0), S(3), CNOT(1,3), S(3), HAD(0), HAD(1), CNOT(3,1), CNOT(3,2)]

Most of the functionality in PyZX works on Graphs instead, which directly represent ZX-diagrams (the drawing function ``zx.draw`` above for instance first converted the circuit into a Graph before drawing it). ZX-diagrams are represented by instances of :class:`~pyzx.graph.base.BaseGraph`. To convert a circuit into a ZX-diagram, simply do::

	g = circuit.to_graph()


Let us use one of the built-in ZX-diagram simplification routines on this ZX-diagram::
	
	>>> zx.clifford_simp(g)  # simplifies the diagram
	>>> g.normalize()  # makes it more presentable
	>>> zx.draw(g)

.. figure::  _static/clifford_simp.png
   :align:   center

   The same circuit, but rewritten into a more compact ZX-diagram. The blue lines represent edges which have a Hadamard gate on them.

Internally, a ZX-diagram is just a graph with some additional data::
	
	>>> print(g)
	Graph(16 vertices, 21 edges)


The simplified ZX-diagram above no longer looks like a circuit. PyZX supplies some methods for turning a Graph back into a circuit::
	
	>>> c = zx.extract_circuit(g.copy())
	>>> zx.draw(c)

.. figure::  _static/clifford_extracted.png
   :align:   center

This extraction procedure is sometimes not as good at keeping the number of two-qubit gates low, and will sometimes increase the size of the circuit. PyZX also supplies some Circuit-level optimisers that more consistently reduce the size of the circuit (but are less powerful)::
	
	>>> c2 = zx.optimize.basic_optimization(c.to_basic_gates())
	>>> zx.draw(c2)

.. figure::  _static/clifford_optimized.png
   :align:   center

To verify that the optimized circuit is still equal to the original we can convert them to numpy tensors and compare equality directly::
	
	>>> zx.compare_tensors(c2,g,preserve_scalar=False)
		True

We can convert circuits into one of several circuit description languages, such as QASM::
	
	>>> print(c2.to_qasm())
	OPENQASM 2.0;
	include "qelib1.inc";
	qreg q[4];
	rz(0.5*pi) q[1];
	h q[1];
	rz(0.5*pi) q[1];
	cx q[2], q[0];
	h q[2];
	h q[3];
	h q[0];
	cx q[0], q[1];
	sdg q[1];
	cx q[2], q[1];
	cz q[0], q[2];
	h q[2];
	cz q[0], q[3];
	h q[3];
	rz(0.5*pi) q[3];
	h q[0];
	x q[0];
	cx q[1], q[2];
	cx q[2], q[1];
	cx q[1], q[2];


Optimizing random circuits is of course not very useful, so let us do some optimization on a predefined circuit::

	>>> c = zx.Circuit.load('circuits/Fast/mod5_4_before')  # Circuit.load auto-detects the file format
	>>> print(c.gates)  #  This circuit is built out of CCZ gates.
	[NOT(4), HAD(4), CCZ(c1=0,c2=3,t=4), CCZ(c1=2,c2=3,t=4), HAD(4), CNOT(3,4), HAD(4), CCZ(c1=1,c2=2,t=4), HAD(4), CNOT(2,4), HAD(4), CCZ(c1=0,c2=1,t=4), HAD(4), CNOT(1,4), CNOT(0,4)]
	>>> c = c.to_basic_gates()  #  Convert it to the Clifford+T gate set.
	>>> print(c.gates)
	[NOT(4), HAD(4), CNOT(3,4), T*(4), CNOT(0,4), T(4), CNOT(3,4), T*(4), CNOT(0,4), T(3), T(4), HAD(4), CNOT(0,3), T(0), T*(3), CNOT(0,3), HAD(4), CNOT(3,4), T*(4), CNOT(2,4), T(4), CNOT(3,4), T*(4), CNOT(2,4), T(3), T(4), HAD(4), CNOT(2,3), T(2), T*(3), CNOT(2,3), HAD(4), HAD(4), CNOT(3,4), HAD(4), CNOT(2,4), T*(4), CNOT(1,4), T(4), CNOT(2,4), T*(4), CNOT(1,4), T(2), T(4), HAD(4), CNOT(1,2), T(1), T*(2), CNOT(1,2), HAD(4), HAD(4), CNOT(2,4), HAD(4), CNOT(1,4), T*(4), CNOT(0,4), T(4), CNOT(1,4), T*(4), CNOT(0,4), T(1), T(4), HAD(4), CNOT(0,1), T(0), T*(1), CNOT(0,1), HAD(4), HAD(4), CNOT(1,4), CNOT(0,4)]
	>>> print(c.stats())
	Circuit mod5_4_before on 5 qubits with 71 gates.
		28 is the T-count
		43 Cliffords among which
		28 2-qubit gates and 14 Hadamard gates.
	>>> g = c.to_graph()
	>>> print(g)
	Graph(109 vertices, 132 edges)
	>>> zx.simplify.full_reduce(g)  # Simplify the ZX-graph
	>>> print(g)
	Graph(31 vertices, 38 edges)
	>>> c2 = zx.extract_circuit(g).to_basic_gates()  # Turn graph back into circuit
	>>> print(c2.stats())
	Circuit  on 5 qubits with 42 gates.
		8 is the T-count
		34 Cliffords among which
		24 2-qubit gates and 10 Hadamard gates.
	>>> c3 = zx.optimize.full_optimize(c2)  #  Do some further optimization on the circuit
	>>> print(c3.stats())
	Circuit  on 5 qubits with 27 gates.
		8 is the T-count
		19 Cliffords among which
		14 2-qubit gates and 2 Hadamard gates.

The circuit file-formats supported by ``Circuit.load`` are curently *qasm*, *qc* or *quipper*. 
PyZX can also be run from the command-line for some easy circuit-to-circuit manipulation. In order to optimize a circuit you can run the command::
	
	python -m pyzx opt input_circuit.qasm

For more information regarding the command-line tools, run ``python -m pyzx --help``.

This concludes this tutorial. For more information about the simplification procedures see :ref:`simplify`. 
The different representations of the graphs and circuits is detailed in :ref:`representations`. How to create and modify ZX-diagrams is explained in :ref:`graphs`.