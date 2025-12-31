React D3 Tree is a React component that lets you represent hierarchical data (e.g. family trees, org charts, file directories) as an interactive tree graph with minimal setup, by leveraging D3's tree layout.

Upgrading from v1? Check out the v2 release notes.

Legacy v1 docs

Contents
Installation
Usage
Props
Working with the default Tree
Providing data
Styling Nodes
Styling Links
Event Handlers
Customizing the Tree
renderCustomNodeElement
pathFunc
Providing your own pathFunc
Development
Setup
Hot reloading
Contributors
Installation
npm i --save react-d3-tree
Usage
import React from 'react';
import Tree from 'react-d3-tree';

// This is a simplified example of an org chart with a depth of 2.
// Note how deeper levels are defined recursively via the `children` property.
const orgChart = {
  name: 'CEO',
  children: [
    {
      name: 'Manager',
      attributes: {
        department: 'Production',
      },
      children: [
        {
          name: 'Foreman',
          attributes: {
            department: 'Fabrication',
          },
          children: [
            {
              name: 'Worker',
            },
          ],
        },
        {
          name: 'Foreman',
          attributes: {
            department: 'Assembly',
          },
          children: [
            {
              name: 'Worker',
            },
          ],
        },
      ],
    },
  ],
};

export default function OrgChartTree() {
  return (
    // `<Tree />` will fill width/height of its container; in this case `#treeWrapper`.
    <div id="treeWrapper" style={{ width: '50em', height: '20em' }}>
      <Tree data={orgChart} />
    </div>
  );
}
Props
For details on all props accepted by Tree, check out the TreeProps reference docs.

The only required prop is data, all other props on Tree are optional/pre-defined (see "Default value" on each prop definition).

Working with the default Tree
react-d3-tree provides default implementations for Tree's nodes & links, which are intended to get you up & running with a working tree quickly.

This section is focused on explaining how to provide data, styles and event handlers for the default Tree implementation.

Need more fine-grained control over how nodes & links appear/behave? Check out the Customizing the Tree section below.

Providing data
By default, Tree expects each node object in data to implement the RawNodeDatum interface:

interface RawNodeDatum {
  name: string;
  attributes?: Record<string, string | number | boolean>;
  children?: RawNodeDatum[];
}
The orgChart example in the Usage section above is an example of this:

Every node has at least a name. This is rendered as the node's primary label.
Some nodes have attributes defined (the CEO node does not). The key-value pairs in attributes are rendered as a list of secondary labels.
Nodes can have further RawNodeDatum objects nested inside them via the children key, creating a hierarchy from which the tree graph can be generated.
Styling Nodes
Tree provides the following props to style different types of nodes, all of which use an SVG circle by default:

rootNodeClassName - applied to the root node.
branchNodeClassName - applied to any node with 1+ children.
leafNodeClassName - applied to any node without children.
To visually distinguish these three types of nodes from each other by color, we could provide each with their own class:

/* custom-tree.css */

.node__root > circle {
  fill: red;
}

.node__branch > circle {
  fill: yellow;
}

.node__leaf > circle {
  fill: green;
  /* Let's also make the radius of leaf nodes larger */
  r: 40;
}
import React from 'react';
import Tree from 'react-d3-tree';
import './custom-tree.css';

// ...

export default function StyledNodesTree() {
  return (
    <div id="treeWrapper" style={{ width: '50em', height: '20em' }}>
      <Tree
        data={data}
        rootNodeClassName="node__root"
        branchNodeClassName="node__branch"
        leafNodeClassName="node__leaf"
      />
    </div>
  );
}
For more details on the className props for nodes, see the TreeProps reference docs.

Styling Links
Tree provides the pathClassFunc property to pass additional classNames to every link to be rendered.

Each link calls pathClassFunc with its own TreeLinkDatum and the tree's current orientation. Tree expects pathClassFunc to return a className string.

function StyledLinksTree() {
  const getDynamicPathClass = ({ source, target }, orientation) => {
    if (!target.children) {
      // Target node has no children -> this link leads to a leaf node.
      return 'link__to-leaf';
    }

    // Style it as a link connecting two branch nodes by default.
    return 'link__to-branch';
  };

  return (
    <Tree
      data={data}
      // Statically apply same className(s) to all links
      pathClassFunc={() => 'custom-link'}
      // Want to apply multiple static classes? `Array.join` is your friend :)
      pathClassFunc={() => ['custom-link', 'extra-custom-link'].join(' ')}
      // Dynamically determine which `className` to pass based on the link's properties.
      pathClassFunc={getDynamicPathClass}
    />
  );
}
For more details, see the PathClassFunction reference docs.

Event Handlers
Tree exposes the following event handler callbacks by default:

onLinkClick
onLinkMouseOut
onLinkMouseOver
onNodeClick
onNodeMouseOut
onNodeMouseOver
Note: Nodes are expanded/collapsed whenever onNodeClick fires. To prevent this, set the collapsible prop to false.
onNodeClick will still fire, but it will not change the target node's expanded/collapsed state.

Customizing the Tree
renderCustomNodeElement
The renderCustomNodeElement prop accepts a custom render function that will be used for every node in the tree.

Cases where you may find rendering your own Node element useful include:

Using a different SVG tag for your nodes (instead of the default <circle>) - Example (codesandbox.io)
Gaining fine-grained control over event handling (e.g. to implement events not covered by the default API) - Example (codesandbox.io)
Building richer & more complex nodes/labels by leveraging the foreignObject tag to render HTML inside the SVG namespace - Example (codesandbox.io)
pathFunc
The pathFunc prop accepts a predefined PathFunctionOption enum or a user-defined PathFunction.

By changing or providing your own pathFunc, you are able to change how links between nodes of the tree (which are SVG path tags under the hood) are drawn.

The currently available enums are:

diagonal (default)
elbow
straight
step
Want to see how each option looks? Try them out on the playground.

Providing your own pathFunc
If none of the available path functions suit your needs, you're also able to provide a custom PathFunction:

function CustomPathFuncTree() {
  const straightPathFunc = (linkDatum, orientation) => {
    const { source, target } = linkDatum;
    return orientation === 'horizontal'
      ? `M${source.y},${source.x}L${target.y},${target.x}`
      : `M${source.x},${source.y}L${target.x},${target.y}`;
  };

  return (
    <Tree
      data={data}
      // Passing `straight` function as a custom `PathFunction`.
      pathFunc={straightPathFunc}
    />
  );
}
For more details, see the PathFunction reference docs.

Development
Setup
To set up react-d3-tree for local development, clone the repo and follow the steps below:

# 1. Set up the library, create a reference to it for symlinking.
cd react-d3-tree
npm i
npm link

# 2. Set up the demo/playground, symlink to the local copy of `react-d3-tree`.
cd demo
npm i
npm link react-d3-tree
Tip: If you'd prefer to use your own app for development instead of the demo, simply run npm link react-d3-tree in your app's root folder instead of the demo's :)

Hot reloading
npm run build:watch
If you're using react-d3-tree/demo for development, open up another terminal window in the demo directory and call:

npm start
