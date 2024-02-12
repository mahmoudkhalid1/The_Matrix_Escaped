package code;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.HashSet;

import code.Location;
import code.TreeNode;
import code.Queue.*;

public abstract class GeneralSearch {

	// habdyn el cost

	// general search method==> returns the string required in the pdf
	public static String generalSearch(String grid, String strategy, boolean visualize) {
		if (!(strategy.equals("ID"))) {
			String result = "";
			// initialize everything as this is the start
			ArrayList<TreeNode> prevNodes = new ArrayList<TreeNode>();
			String[] splitted = grid.split(";");
			// get Neo's starting position from the given grid
			String[] preNeo = splitted[2].split(",");
			Location neo = new Location(Integer.parseInt(preNeo[0]), Integer.parseInt(preNeo[1]));
			// array to store the damages of the carried hostages and keep track of their
			// number
			ArrayList<Integer> carried = new ArrayList<Integer>();

			// create initial starting node
			TreeNode start = new TreeNode(null, prevNodes, neo, 0, grid, 0, 0, "Start", 0, 0, 0, carried);
			double startAcost = calculateActualCost(start, "Start");
			double startHCost;
			if (strategy.equals("GR1") || strategy.equals("AS1")) {
				startHCost = calculateH(start, "Start");
				System.out.println(startHCost);
			} else {
				startHCost = calculatehCost2(start, "Start");
			}
			start.actualCost = startAcost;
			start.hCost = startHCost;
			PreNode startPre = new PreNode("Start", neo, start, startAcost, startHCost, strategy);
			//hashset to compare repeated states
			HashSet prevNodesHash = new HashSet();
			String state = start.myLoc.x + ";" + start.myLoc.y + ";"
					+ getGridWithoutDamages(start.grid) + "," 
					+ start.carried.size() + ","
					+ start.kills + ","
					+ start.deaths + ","
					+ start.neoDamage;
					
			prevNodesHash.add(state);
			// add to the array of previous nodes to check for repeated states
			prevNodes.add(start);

			if (visualize) {
				System.out.println("Starting the game with search strategy: " + strategy);
				System.out.println("Neo is at cell: " + neo.x + "  " + neo.y);
			}

			// get the possible actions for the starting node
			ArrayList<String> possibleActions = getPossibleActions(start);

			// try to favor carry and kill mutant
			// create the queue for search
			Queue queue = null;
			PrQ pqueue = null;
			switch (strategy) {
			case ("DF"):
				queue = new DFQueue();
				ArrayList<String> temp = new ArrayList<String>();
				// organize possible actions accordingly
				for (int i = 0; i < possibleActions.size(); i++) {
					String pa = possibleActions.get(i);
					if (pa.contains("kill")) {
						temp.add(possibleActions.remove(i));
						i--;
					} else if (pa.contains("carry")) {
						temp.add(possibleActions.remove(i));
						i--;
					} else if (pa.contains("drop")) {
						temp.add(possibleActions.remove(i));
						i--;
					}
				}
				for (int i = 0; i < temp.size(); i++) {
					possibleActions.add(temp.get(i));
				}
				temp = new ArrayList<String>();
				break;
			case ("BF"):
				queue = new BFQueue();
				break;
			case ("UC"):
				pqueue = new PrQ();
				break;
			case ("GR1"):
				pqueue = new PrQ();
				break;
			case ("GR2"):
				pqueue = new PrQ();
				break;
			case ("AS1"):
				pqueue = new PrQ();
				break;
			case ("AS2"):
				pqueue = new PrQ();
				break;
			default:
				// temporary
				queue = new DFQueue();
				break;
			}

			for (int i = 0; i < possibleActions.size(); i++) {
				if (!(possibleActions.get(i).contains("kill"))) {
					String[] pa = possibleActions.get(i).split(",");
					Location affected = new Location(Integer.parseInt(pa[1]), Integer.parseInt(pa[2]));
					double actualCost = calculateActualCost(start, pa[0]);
					double hCost;
					if (strategy.equals("GR1") || strategy.equals("AS1")) {
						hCost = calculateH(start, possibleActions.get(i));
					} else {
						hCost = calculatehCost2(start, possibleActions.get(i));
					}
					PreNode pn = new PreNode(pa[0], affected, start, actualCost, hCost, strategy);
					if (strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID")) {
						queue.enqueue(pn);
					} else {
						pqueue.enqueue(pn);
					}

				} else {
					// enqueue once and handle number of kills inside update
					String[] pk = possibleActions.get(i).split(";");
					String[] pa = pk[0].split(",");
					double actualCost = calculateActualCost(start, possibleActions.get(i));
					double hCost;
					if (strategy.equals("GR1") || strategy.equals("AS1")) {
						hCost = calculateH(start, possibleActions.get(i));
					} else {
						hCost = calculatehCost2(start, possibleActions.get(i));
					}
					PreNode pn = new PreNode(pa[0], neo, start, actualCost, hCost, strategy);
					if (strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID")) {
						queue.enqueue(pn);
					} else {
						pqueue.enqueue(pn);
					}
				}
			}
			boolean failed = false;
			while (((strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID"))
					&& (!queue.queue.isEmpty()))
					|| ((strategy.equals("UC") || strategy.contains("GR") || strategy.contains("AS"))
							&& (!pqueue.queue.isEmpty()))) {
				if (visualize) {
					System.out.println("Removing a PreNode from the queue ");
				}
				PreNode frontPreNode;
				if (strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID")) {
					frontPreNode = queue.dequeue();
				} else {
					frontPreNode = pqueue.dequeue();
				}

				if (visualize) {
					System.out.println("The prenode: " + frontPreNode.action);
				}
				boolean repeated = true;
				TreeNode frontTreeNode = update(
						frontPreNode.action + "," + frontPreNode.affectedCell.x + "," + frontPreNode.affectedCell.y,
						frontPreNode.prevNode, prevNodes);
				frontTreeNode.actualCost = frontPreNode.actualCost;
				frontTreeNode.hCost = frontPreNode.hCost;
				
				// check if gameOver
				if (gameOver(frontTreeNode.neoDamage)) {
					System.out.println("Game Over at this path");
					continue;
				}

				// check if goal
				// return the requirements of solve
				if (isItGoal(frontTreeNode)) {
					result = "";
					System.out.println("Daret ya syaaaa3");
					double pathCost = 0;
					ArrayList<String> goalPath = new ArrayList<String>();
					TreeNode p = frontTreeNode;
					while (p != null) {
						if (visualize) {
							System.out.println(p.operator + "   ," + p.actualCost + "a  , "
										+ p.hCost + "h,   " + p.grid);
						}
						goalPath.add(p.operator);
						pathCost += p.actualCost;
						p = p.parent;
					}
					for (int z = goalPath.size() - 2; z >= 0; z--) {
						if (z == 0) {
							result += goalPath.get(z);
						} else {
							result += goalPath.get(z) + ",";
						}
					}
					if (visualize) {
						System.out.println("path cost: " + pathCost);
					}
					System.out.println("path cost: " + pathCost);
					return result += ";" + frontTreeNode.deaths + ";" + frontTreeNode.kills + ";" + prevNodes.size();
				}
				
				state = frontTreeNode.myLoc.x + ";" + frontTreeNode.myLoc.y + ";"
						+ getGridWithoutDamages(frontTreeNode.grid) + "," 
						+ frontTreeNode.carried.size() + ","
						+ frontTreeNode.kills + ","
						+ frontTreeNode.deaths + ","
						+ frontTreeNode.neoDamage;
						
				repeated = prevNodesHash.add(state);

				if (repeated == false) {
					if (visualize) {
						System.out.println("Following this action will lead to a repeated state so I ignored it ");
					}
					continue;
				}
				if (visualize) {

					System.out.println(
							"Neo was at cell: " + frontPreNode.prevNode.myLoc.x + "  " + frontPreNode.prevNode.myLoc.y);
					System.out.println(" After applying the action: " + frontPreNode.action + " Neo is now at cell: "
							+ frontTreeNode.myLoc.x + "  " + frontTreeNode.myLoc.y);
					System.out.println(" Neo's damage is now: " + frontTreeNode.neoDamage);
					System.out.println(" Number of Kills: " + frontTreeNode.kills);
					System.out.println(" Number of Deaths: " + frontTreeNode.deaths);
					System.out.println(" Neo is carrying: " + frontTreeNode.carried.size() + " hostages");
					System.out.println(" The grid is now: " + frontTreeNode.grid);
				}

				prevNodes.add(frontTreeNode);
				possibleActions = getPossibleActions(frontTreeNode);
				if (strategy.equals("DF")) {
					ArrayList<String> temp = new ArrayList<String>();
					// organize possible actions accordingly
					for (int i = 0; i < possibleActions.size(); i++) {
						String pa = possibleActions.get(i);
						if (pa.contains("kill")) {
							temp.add(possibleActions.remove(i));
							i--;
						} else if (pa.contains("carry")) {
							temp.add(possibleActions.remove(i));
							i--;
						} else if (pa.contains("drop")) {
							temp.add(possibleActions.remove(i));
							i--;
						}
					}
					for (int i = 0; i < temp.size(); i++) {
						possibleActions.add(temp.get(i));
					}
					temp = new ArrayList<String>();
				}
				for (int i = 0; i < possibleActions.size(); i++) {
					if (!(possibleActions.get(i).contains("kill"))) {
						String[] pa = possibleActions.get(i).split(",");
						Location affected = new Location(Integer.parseInt(pa[1]), Integer.parseInt(pa[2]));
						double actualCost = calculateActualCost(frontTreeNode, pa[0]);
						double hCost;
						if (strategy.equals("GR1") || strategy.equals("AS1")) {
							hCost = calculateH(frontTreeNode, possibleActions.get(i));
						} else {
							hCost = calculatehCost2(frontTreeNode, possibleActions.get(i));
						}
						PreNode pn = new PreNode(pa[0], affected, frontTreeNode, actualCost, hCost, strategy);
						if (strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID")) {
							queue.enqueue(pn);
						} else {
							pqueue.enqueue(pn);
						}

					} else {
						// enqueue once and handle number of kills inside update
						String[] pk = possibleActions.get(i).split(";");
						String[] pa = pk[0].split(",");
						double actualCost = calculateActualCost(frontTreeNode, possibleActions.get(i));
						double hCost;
						if (strategy.equals("GR1") || strategy.equals("AS1")) {
							hCost = calculateH(frontTreeNode, possibleActions.get(i));
						} else {
							hCost = calculatehCost2(frontTreeNode, possibleActions.get(i));
						}
						PreNode pn = new PreNode(pa[0], frontTreeNode.myLoc, frontTreeNode, actualCost, hCost,
								strategy);
						if (strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID")) {
							queue.enqueue(pn);
						} else {
							pqueue.enqueue(pn);
						}
					}
				}
				if (((strategy.equals("DF") || strategy.equals("BF") || strategy.equals("ID"))
						&& (queue.queue.isEmpty()))
						|| ((strategy.equals("UC") || strategy.contains("GR") || strategy.contains("AS"))
								&& (pqueue.queue.isEmpty()))) {
					return "No Solution";
				}
			}
			return "No Solution";
		} else {
			String result = "";

			int k = 0;
			while (true) {
				// initialize everything as this is the start
				//System.out.println("Start again");
				ArrayList<TreeNode> prevNodes = new ArrayList<TreeNode>();
				ArrayList<PreNode> preNodes = new ArrayList<PreNode>();
				String[] splitted = grid.split(";");
				// get Neo's starting position from the given grid
				String[] preNeo = splitted[2].split(",");
				Location neo = new Location(Integer.parseInt(preNeo[0]), Integer.parseInt(preNeo[1]));
				// array to store the damages of the carried hostages and keep track of their
				// number
				ArrayList<Integer> carried = new ArrayList<Integer>();
				// create initial starting node
				TreeNode start = new TreeNode(null, prevNodes, neo, 0, grid, 0, 0, "Start", 0, 0, 0, carried);
				PreNode startPre = new PreNode("Start", neo, start, 0, 0, strategy);
				HashSet prevNodesHash = new HashSet();
				String state = start.myLoc.x + ";" + start.myLoc.y + ";"
						+ getGridWithoutDamages(start.grid) + "," 
						+ start.carried.size() + ","
						+ start.kills + ","
						+ start.deaths + ","
						+ start.neoDamage;
						
				prevNodesHash.add(state);
				// add to the array of previous nodes to check for repeated states
				prevNodes.add(start);

				if (visualize) {
					System.out.println("Starting the game with search strategy: " + strategy);
					System.out.println("Neo is at cell: " + neo.x + "  " + neo.y);
				}

				// get the possible actions for the starting node
				ArrayList<String> possibleActions = getPossibleActions(start);

				// try to favor carry and kill mutant
				// create the queue for search
				Queue queue = new DFQueue();

				ArrayList<String> temp = new ArrayList<String>();
				// organize possible actions accordingly
				for (int i = 0; i < possibleActions.size(); i++) {
					String pa = possibleActions.get(i);
					if (pa.contains("kill")) {
						temp.add(possibleActions.remove(i));
						i--;
					} else if (pa.contains("carry")) {
						temp.add(possibleActions.remove(i));
						i--;
					} else if (pa.contains("drop")) {
						temp.add(possibleActions.remove(i));
						i--;
					}
				}
				for (int i = 0; i < temp.size(); i++) {
					possibleActions.add(temp.get(i));
				}
				temp = new ArrayList<String>();

				boolean stop = false;
				for (int j = 0; j < possibleActions.size(); j++) {
					if (!(possibleActions.get(j).contains("kill"))) {
						String[] pa = possibleActions.get(j).split(",");
						Location affected = new Location(Integer.parseInt(pa[1]), Integer.parseInt(pa[2]));
						PreNode pn = new PreNode(pa[0], affected, start, 0, 0, strategy);
						if (pn.depth > k) {
							stop = true;
							break;
						}
						queue.enqueue(pn);
					} else {
						// enqueue once and handle number of kills inside update
						String[] pk = possibleActions.get(j).split(";");
						String[] pa = pk[0].split(",");
						PreNode pn = new PreNode(pa[0], neo, start, 0, 0, strategy);
						if (pn.depth > k) {
							stop = true;
							break;
						}
						queue.enqueue(pn);
					}
				}
				if (stop) {
					k++;
					continue;
				}
				if (visualize) {
					System.out.println(
							"The possible action(s) available at this cell is/are (as ordered in the queue): ");
					queue.display();
				}

				boolean failed = false;
				while (!queue.queue.isEmpty()) {
					if (visualize) {
						System.out.println("Removing a PreNode from the queue ");
					}

					PreNode frontPreNode = queue.dequeue();

					if (visualize) {
						System.out.println("The prenode: " + frontPreNode.action + frontPreNode.depth);
					}
					boolean repeated = true;
					TreeNode frontTreeNode = update(
							frontPreNode.action + "," + frontPreNode.affectedCell.x + "," + frontPreNode.affectedCell.y,
							frontPreNode.prevNode, prevNodes);

					// check if gameOver
					if (gameOver(frontTreeNode.neoDamage)) {
						System.out.println("Game Over at this path");
						continue;
					}

					// check if goal
					// return the requirements of solve
					if (isItGoal(frontTreeNode)) {
						result = "";
						System.out.println("Daret ya syaaaa3");
						ArrayList<String> goalPath = new ArrayList<String>();
						TreeNode p = frontTreeNode;
						while (p != null) {
							System.out.println(p.operator);
							goalPath.add(p.operator);
							p = p.parent;
						}
						for (int z = goalPath.size() - 2; z >= 0; z--) {
							if (z == 0) {
								result += goalPath.get(z);
							} else {
								result += goalPath.get(z) + ",";
							}
						}
						return result += ";" + frontTreeNode.deaths + ";" + frontTreeNode.kills + ";"
								+ prevNodes.size();
					}

					state = frontTreeNode.myLoc.x + ";" + frontTreeNode.myLoc.y + ";"
							+ getGridWithoutDamages(frontTreeNode.grid) + "," 
							+ frontTreeNode.carried.size() + ","
							+ frontTreeNode.kills + ","
							+ frontTreeNode.deaths + ","
							+ frontTreeNode.neoDamage;
							
					repeated = prevNodesHash.add(state);
					if (repeated == false) {
						if (visualize) {
							System.out.println("Following this action will lead to a repeated state so I ignored it ");
						}
						continue;
					}
					if (visualize) {
						System.out.println("Neo was at cell: " + frontPreNode.prevNode.myLoc.x + "  "
								+ frontPreNode.prevNode.myLoc.y);
						System.out.println(" After applying the action: " + frontPreNode.action
								+ " Neo is now at cell: " + frontTreeNode.myLoc.x + "  " + frontTreeNode.myLoc.y);
						System.out.println(" Neo's damage is now: " + frontTreeNode.neoDamage);
						System.out.println(" Number of Kills: " + frontTreeNode.kills);
						System.out.println(" Number of Deaths: " + frontTreeNode.deaths);
						System.out.println(" Neo is carrying: " + frontTreeNode.carried.size() + " hostages");
						System.out.println(" The grid is now: " + frontTreeNode.grid);
					}
					result += frontPreNode.action;
					prevNodes.add(frontTreeNode);
					possibleActions = getPossibleActions(frontTreeNode);

					ArrayList<String> temp2 = new ArrayList<String>();
					// organize possible actions accordingly
					for (int i = 0; i < possibleActions.size(); i++) {
						String pa = possibleActions.get(i);
						if (pa.contains("kill")) {
							temp2.add(possibleActions.remove(i));
							i--;
						} else if (pa.contains("carry")) {
							temp2.add(possibleActions.remove(i));
							i--;
						} else if (pa.contains("drop")) {
							temp2.add(possibleActions.remove(i));
							i--;
						}
					}
					for (int i = 0; i < temp2.size(); i++) {
						possibleActions.add(temp2.get(i));
					}
					temp2 = new ArrayList<String>();

					for (int j = 0; j < possibleActions.size(); j++) {
						if (!(possibleActions.get(j).contains("kill"))) {
							String[] pa = possibleActions.get(j).split(",");
							Location affected = new Location(Integer.parseInt(pa[1]), Integer.parseInt(pa[2]));
							PreNode pn = new PreNode(pa[0], affected, frontTreeNode, 0, 0, strategy);
							if (pn.depth > k) {
								stop = true;
								break;
							}
							queue.enqueue(pn);
						} else {
							// enqueue once and handle number of kills inside update
							String[] pk = possibleActions.get(j).split(";");
							String[] pa = pk[0].split(",");
							PreNode pn = new PreNode(pa[0], frontTreeNode.myLoc, frontTreeNode, 0, 0, strategy);
							if (pn.depth > k) {
								stop = true;
								break;
							}
							queue.enqueue(pn);
						}
					}
					if (stop) {
						break;
					}

					if (queue.queue.isEmpty()) {
						System.out.println("failed");
						result += "No Solution";
						return "No Solution";
					}
				}
				k++;
			}
		}

	}

	// returns true if neo is dead, false otherwise
	public static boolean gameOver(int neoD) {
		boolean gameOver = false;
		if (neoD >= 100) {
			gameOver = true;
		}
		return gameOver;
	}

	// compares the grids of the two nodes in comparison to help check for repeated
	// states
	// TO-DO: this might be tricky if we needed the damages as a difference
	public static boolean compareGrids(String grid1, String grid2) {
		boolean similar = false;
		// if same number of agents and hostages and pills then similar
		String[] splitted1 = grid1.split(";");
		String[] splitted2 = grid2.split(";");
		String[] agents1 = splitted1[4].split(",");
		String[] agents2 = splitted2[4].split(",");
		if (agents1.length == agents2.length) {
			// check pills
			String[] pills1 = splitted1[5].split(",");
			String[] pills2 = splitted2[5].split(",");
			if (pills1.length == pills2.length) {
				// check hostages
				String[] hos1;
				// this check is needed because if there are no more hostages in the grid
				// the last part of the grid will not appear after split
				if (splitted1.length <= 7) {
					// no more hostages
					hos1 = new String[0];
				} else {
					hos1 = splitted1[7].split(",");
				}

				String[] hos2;
				if (splitted2.length <= 7) {
					// no more hostages
					hos2 = new String[0];
				} else {
					hos2 = splitted2[7].split(",");
				}
				if (hos1.length == hos2.length) {
					similar = true;
				} else {
					similar = false;
				}
			} else {
				similar = false;
			}
		} else {
			similar = false;
		}
		return similar;
	}

	
	public static String getGridWithoutDamages(String grid1) {
		String result = "";
		String[] splitted1 = grid1.split(";");
		result += splitted1[0] + splitted1[1] + splitted1[2] + splitted1[3] + splitted1[4] + splitted1[5];
		String[] hos1;
		if (splitted1.length <= 7) {
			// no more hostages
			hos1 = new String[0];
		} else {
			hos1 = splitted1[7].split(",");
		}	
		for (int i = 0; i < hos1.length - 1; i += 3) {
			result += hos1[i] + "," + hos1[i+1];
			if (i < hos1.length - 2) {
				result += ",";
			}
		}
			
		return result;
	}

	// goal test
	public static boolean isItGoal(TreeNode n) {
		boolean goal = false;
		String grid = n.grid;
		String[] splitted = grid.split(";");
		// array that consists of (x,y) of the telephone booth
		String[] telephone = splitted[3].split(",");
		Location tb = new Location(Integer.parseInt(telephone[0]), Integer.parseInt(telephone[1]));
		ArrayList<String> hostages = getHostages(grid);
		ArrayList<String> mutantHostages = getMutantHostages(grid);
		if (mutantHostages.size() == 0 && hostages.size() == 0 && n.carried.size() == 0 && n.myLoc.equals(tb)) {
			goal = true;
		}
		return goal;

	}

	// method to print the content of any array
	public static void printArr(ArrayList<String> arr) {
		for (int i = 0; i < arr.size(); i++) {
			System.out.println(arr.get(i));
		}
		System.out.println(" ");
	}

	// returns arraylist of hostages with their damages
	// each element is in the form of "hosX,hosY,damage"
	public static ArrayList<String> getHostages(String grid) {
		ArrayList<String> result = new ArrayList<String>();
		String[] splitted = grid.split(";");
		// array that contains locations of all the hostages
		String[] hostages;
		if (splitted.length <= 7) {
			hostages = new String[0];
		} else {
			hostages = splitted[7].split(",");
		}

		for (int i = 0; i < hostages.length - 2; i += 3) {
			// store the location and damage in a string
			String temp = "";
			// check the damage to know if mutant or alive
			// if mutant then skip this iteration
			if (Integer.parseInt(hostages[i + 2]) < 100) {
				temp += hostages[i] + "," + hostages[i + 1] + "," + hostages[i + 2];
				result.add(temp);
			}

		}

		return result;
	}

	// returns arraylist of mutant hostages
	// each element is in the form of "hosX,hosY"
	public static ArrayList<String> getMutantHostages(String grid) {
		ArrayList<String> result = new ArrayList<String>();
		String[] splitted = grid.split(";");
		// array that contains locations of all the hostages
		String[] hostages;
		if (splitted.length <= 7) {
			hostages = new String[0];
		} else {
			hostages = splitted[7].split(",");
		}

		for (int i = 0; i < hostages.length - 2; i += 3) {
			// store the location in a string
			String temp = "";
			// check the damage to know if mutant or alive
			// if alive then skip this iteration
			if (Integer.parseInt(hostages[i + 2]) >= 100) {
				// add only the location since we do not need the damage
				temp += hostages[i] + "," + hostages[i + 1];
				result.add(temp);
			}
		}
		return result;
	}

	// method to get the content of the given cell and useful information about
	// content of cell
	// can be used to determine the surroundings of the current cell
	public static String whatInCell(int x, int y, String grid) {
		String result = "";
		// split the grid at ; to extract different categories in the grid
		String[] splitted = grid.split(";");
		// array that consists of (x,y) of the telephone booth
		String[] telephone = splitted[3].split(",");
		// array that contains locations of all the agents
		String[] agents = splitted[4].split(",");
		// array that contains locations of all pills
		String[] pills = splitted[5].split(",");
		// array that contains locations of all the pads
		// this array is always divisible by 4 since the pad comes in pairs
		// (startx; starty; finishx; finishy)
		String[] pads = splitted[6].split(",");
		// array that contains locations of all the hostages
		String[] hostages;
		if (splitted.length <= 7) {
			hostages = new String[0];
		} else {
			hostages = splitted[7].split(",");
		}

		if (x == Integer.parseInt(telephone[0]) && y == Integer.parseInt(telephone[1])) {
			result = "telephone;" + x + ";" + y;
			return result;
		}
		for (int i = 0; i < agents.length - 1; i += 2) {
			if (x == Integer.parseInt(agents[i]) && y == Integer.parseInt(agents[i + 1])) {
				result = "agent;" + x + ";" + y;
				return result;
			}
		}
		for (int i = 0; i < pills.length - 1; i += 2) {
			if (x == Integer.parseInt(pills[i]) && y == Integer.parseInt(pills[i + 1])) {
				result = "pill;" + x + ";" + y;
				return result;
			}
		}
		for (int i = 0; i < pads.length - 1; i += 2) {
			if (x == Integer.parseInt(pads[i]) && y == Integer.parseInt(pads[i + 1])) {
				result = "pad;" + x + ";" + y;
				if ((i / 2) % 2 == 0) {
					result += ";" + pads[i + 2] + ";" + pads[i + 3];
				} else {
					result += ";" + pads[i - 2] + ";" + pads[i - 1];
				}
				// return pair of pads attached together
				return result;
			}
		}
		for (int i = 0; i < hostages.length - 1; i += 3) {
			if (x == Integer.parseInt(hostages[i]) && y == Integer.parseInt(hostages[i + 1])) {
				result = "hostage;" + x + ";" + y + ";" + hostages[i + 2];
				return result;
			}
		}
		if (result == "") {
			result = "Empty";
		}
		return result;
	}

	// method to detect where can I move
	public static ArrayList<String> whereCanIMove(Location current, String grid) {
		ArrayList<String> result = new ArrayList<String>();
		// check all possible directions
		String[] splitted = grid.split(";");
		String[] dimensions = splitted[0].split(",");
		int rows = Integer.parseInt(dimensions[0]);
		int cols = Integer.parseInt(dimensions[1]);
		boolean up = false;
		boolean right = false;
		boolean down = false;
		boolean left = false;
		// check if I can go left
		if (current.y > 0) {
			// check if there is an agent or a mutant agent or if there is a hostage check
			// the damage
			String upCellComponent = whatInCell(current.x, current.y - 1, grid);
			if (!(upCellComponent.contains("agent"))) {
				// check if there is a mutant hostage or the hostage is about to become mutant
				if (upCellComponent.contains("hostage")) {
					String[] hos = upCellComponent.split(";");
					if (Integer.parseInt(hos[3]) < 98) {
						left = true;
					}
				} else {
					left = true;
				}
			}
		}
		// check if I can go down
		if (current.x < rows - 1) {
			// check if there is an agent or a mutant agent
			String rightCellComponent = whatInCell(current.x + 1, current.y, grid);
			if (!(rightCellComponent.contains("agent"))) {
				// check if there is a mutant hostage
				if (rightCellComponent.contains("hostage")) {
					String[] hos = rightCellComponent.split(";");
					if (Integer.parseInt(hos[3]) < 98) {
						down = true;
					}
				} else {
					down = true;
				}
			}
		}
		// check if I can go right
		if (current.y < cols - 1) {
			// check if there is an agent or a mutant agent
			String downCellComponent = whatInCell(current.x, current.y + 1, grid);
			if (!(downCellComponent.contains("agent"))) {
				// check if there is a mutant hostage
				if (downCellComponent.contains("hostage")) {
					String[] hos = downCellComponent.split(";");
					if (Integer.parseInt(hos[3]) < 98) {
						right = true;
					}
				} else {
					right = true;
				}
			}
		}
		// check if I can go Up
		if (current.x > 0) {
			// check if there is an agent or a mutant agent
			String leftCellComponent = whatInCell(current.x - 1, current.y, grid);
			if (!(leftCellComponent.contains("agent"))) {
				// check if there is a mutant hostage
				if (leftCellComponent.contains("hostage")) {
					String[] hos = leftCellComponent.split(";");
					if (Integer.parseInt(hos[3]) < 98) {
						up = true;
					}
				} else {
					up = true;
				}
			}
		}
		if (up) {
			result.add("up");
		}
		if (right) {
			result.add("right");
		}
		if (down) {
			result.add("down");
		}
		if (left) {
			result.add("left");
		}

		return result;
	}

	public static ArrayList<String> getPossibleKills(TreeNode node) {
		ArrayList<String> result = new ArrayList<String>();
		// check all possible directions
		String[] splitted = node.grid.split(";");
		String[] dimensions = splitted[0].split(",");
		int rows = Integer.parseInt(dimensions[0]);
		int cols = Integer.parseInt(dimensions[1]);
		boolean kill = true;
		// check if I am in a cell with a hostage of damage 98 or 99 and if yes then do
		// not perform kill action
		String currentCellComponent = whatInCell(node.myLoc.x, node.myLoc.y, node.grid);
		if (currentCellComponent.contains("hostage")) {
			String[] h = currentCellComponent.split(";");
			if (Integer.parseInt(h[3]) < 98) {
				kill = true;
			} else {
				kill = false;
			}
		}
		// check if I can kill left
		if (node.myLoc.y > 0) {
			// check if there is an agent or a mutant agent or if there is a hostage check
			// the damage
			String upCellComponent = whatInCell(node.myLoc.x, node.myLoc.y - 1, node.grid);
			if (!(upCellComponent.contains("agent"))) {
				// check if there is a mutant hostage or the hostage is about to become mutant
				if (upCellComponent.contains("hostage")) {
					String[] hos = upCellComponent.split(";");
					if ((Integer.parseInt(hos[3]) >= 100) && kill) {
						result.add("KillLeft");
					}
				}
			} else if (kill) {
				result.add("KillLeft");
			}
		}
		// check if I can go down
		if (node.myLoc.x < rows - 1) {
			// check if there is an agent or a mutant agent
			String rightCellComponent = whatInCell(node.myLoc.x + 1, node.myLoc.y, node.grid);
			if (!(rightCellComponent.contains("agent"))) {
				// check if there is a mutant hostage
				if (rightCellComponent.contains("hostage")) {
					String[] hos = rightCellComponent.split(";");
					if ((Integer.parseInt(hos[3]) >= 100) && kill) {
						result.add("KillDown");
					}
				}
			} else if (kill) {
				result.add("KillDown");
			}
		}
		// check if I can go right
		if (node.myLoc.y < cols - 1) {
			// check if there is an agent or a mutant agent
			String downCellComponent = whatInCell(node.myLoc.x, node.myLoc.y + 1, node.grid);
			if (!(downCellComponent.contains("agent"))) {
				// check if there is a mutant hostage
				if (downCellComponent.contains("hostage")) {
					String[] hos = downCellComponent.split(";");
					if ((Integer.parseInt(hos[3]) >= 100) && kill) {
						result.add("KillRight");
					}
				}
			} else if (kill) {
				result.add("KillRight");
			}
		}
		// check if I can go Up
		if (node.myLoc.x > 0) {
			// check if there is an agent or a mutant agent
			String leftCellComponent = whatInCell(node.myLoc.x - 1, node.myLoc.y, node.grid);
			if (!(leftCellComponent.contains("agent"))) {
				// check if there is a mutant hostage
				if (leftCellComponent.contains("hostage")) {
					String[] hos = leftCellComponent.split(";");
					if ((Integer.parseInt(hos[3]) >= 100) && kill) {
						result.add("KillUp");
					}
				}
			} else if (kill) {
				result.add("KillUp");
			}
		}
		return result;
	}

	// method to detect possible actions and insert in queue
	// this method returns array of strings of the following format
	// "ActionName", "affectedLocation"
	public static ArrayList<String> getPossibleActions(TreeNode node) {
		ArrayList<String> result = new ArrayList<String>();
		String[] splitted = node.grid.split(";");
		int cAllowed = Integer.parseInt(splitted[1]);
		// check the current cell components
		String currentCellComponent = whatInCell(node.myLoc.x, node.myLoc.y, node.grid);
		if (currentCellComponent.contains("telephone")) {
			// check before drop if there is a hostage to drop
			if (node.carried.size() > 0) {
				// System.out.println(node.depth);
				result.add("drop," + node.myLoc.x + "," + node.myLoc.y);
			}
		}
		if (currentCellComponent.contains("hostage")) {
			String[] hos = currentCellComponent.split(";");
			// cannot be a mutant hostage because we do not add this option to happen during
			// movement
			// in the update method we check if we can actually perform the carry or not
			if (node.carried.size() + 1 <= cAllowed && Integer.parseInt(hos[3]) < 100) {
				result.add("carry," + node.myLoc.x + "," + node.myLoc.y);
			}

		}
		if (currentCellComponent.contains("pill")) {
			// either take it or leave it
			result.add("takePill," + node.myLoc.x + "," + node.myLoc.y);
		}

		if (currentCellComponent.contains("pad")) {
			// get the go-to pad from the string
			String[] padData = currentCellComponent.split(";");
			// we know that the current cell is the first pad in the string so ignore it
			int fx = Integer.parseInt(padData[3]);
			int fy = Integer.parseInt(padData[4]);
			result.add("fly," + fx + "," + fy);
		}

		// get the possible movements
		Location current = new Location(node.myLoc.x, node.myLoc.y);
		ArrayList<String> movements = whereCanIMove(current, node.grid);
		if (!(movements.isEmpty())) {
			for (int i = 0; i < movements.size(); i++) {
				if (movements.get(i) == "up") {
					result.add("up," + (current.x - 1) + "," + current.y);
				} else if (movements.get(i) == "right") {
					result.add("right," + current.x + "," + (current.y + 1));
				} else if (movements.get(i) == "down") {
					result.add("down," + (current.x + 1) + "," + current.y);
				} else if (movements.get(i) == "left") {
					result.add("left," + current.x + "," + (current.y - 1));
				}
			}
		}

		ArrayList<String> kills = getPossibleKills(node);
		String toKill = "";
		if (!(kills.isEmpty())) {
			for (int i = 0; i < kills.size(); i++) {
				if (kills.get(i) == "KillUp") {
					toKill += "kill," + (current.x - 1) + "," + current.y;
				} else if (kills.get(i) == "KillRight") {
					toKill += "kill," + current.x + "," + (current.y + 1);
				} else if (kills.get(i) == "KillDown") {
					toKill += "kill," + (current.x + 1) + "," + current.y;
				} else if (kills.get(i) == "KillLeft") {
					toKill += "kill," + current.x + "," + (current.y - 1);
				}
				if (i < kills.size() - 1) {
					toKill += ";";
				}
			}
			result.add(toKill);
		}
		return result;
	}

	// updates everything needed according to the action taken
	// returns the new node resulting from the update
	public static TreeNode update(String action, TreeNode prevNode, ArrayList<TreeNode> prevNodes) {
		// we assume that Neo starts with 0 damage
		// resultant grid
		String result = "";
		// split the grid at ; to extract different categories in the grid
		String[] splitted = prevNode.grid.split(";");
		// since the dimensions and neo's initial position and TB position are constant,
		// append to result
		result += splitted[0] + ";" + splitted[1] + ";" + splitted[2] + ";" + splitted[3] + ";";
		// array that contains locations of all the agents
		String[] agents = splitted[4].split(",");
		// array that contains the locations of all mutant hostages
		ArrayList<String> mutantHostages = getMutantHostages(prevNode.grid);
		// array that contains locations of all the pills
		String[] pills = splitted[5].split(",");
		// array that contains locations of all the pads
		// this array is always divisible by 4 since the pad comes in pairs
		// (startx, starty, finishx, finishy)
		String pads = splitted[6];
		// array that contains locations of all the hostages
		ArrayList<String> hostages = getHostages(prevNode.grid);
		ArrayList<Integer> carried = new ArrayList<Integer>();

		for (int i = 0; i < prevNode.carried.size(); i++) {
			carried.add(prevNode.carried.get(i));
		}

		int deaths = prevNode.deaths;
		int kills = prevNode.kills;
		int neoD = prevNode.neoDamage;
		Location newLocation = prevNode.myLoc;
		// get the affected location from action
		String[] actionDetails = action.split(",");
		Location moveTo = new Location(Integer.parseInt(actionDetails[1]), Integer.parseInt(actionDetails[2]));

		// check the action performed and accordingly update the grid or damage
		switch (actionDetails[0]) {
		case ("kill"):
			// check if I am in a cell with a hostage of damage 98 or 99 and if yes then do
			// not perform this action
			String currentCellComponent = whatInCell(prevNode.myLoc.x, prevNode.myLoc.y, prevNode.grid);
			if (currentCellComponent.contains("hostage")) {
				// check the damage
				String[] h = currentCellComponent.split(";");
				if (Integer.parseInt(h[3]) < 98) {
					// check how many kills should be performed
					ArrayList<String> toKill = getPossibleKills(prevNode);
					neoD += 20;
					for (int w = 0; w < toKill.size(); w++) {
						// perform the kill
						// increment the number of killings by 1
						kills++;
						// increment Neo's damage by 20

						int kx = -1;
						int ky = -1;
						if (toKill.get(w).contains("KillUp")) {
							kx = prevNode.myLoc.x - 1;
							ky = prevNode.myLoc.y;
						} else if (toKill.get(w).contains("KillRight")) {
							kx = prevNode.myLoc.x;
							ky = prevNode.myLoc.y + 1;
						} else if (toKill.get(w).contains("KillDown")) {
							kx = prevNode.myLoc.x + 1;
							ky = prevNode.myLoc.y;
						} else if (toKill.get(w).contains("KillLeft")) {
							kx = prevNode.myLoc.x;
							ky = prevNode.myLoc.y - 1;
						}
						// update grid by removing the killed agent or mutant hostage
						for (int i = 0; i < agents.length - 1; i += 2) {
							if (kx == Integer.parseInt(agents[i]) && ky == Integer.parseInt(agents[i + 1])) {
								// replace the x and y with negatives so that when we combine the string again
								// we can know that this is not valid
								agents[i] = "-1";
								agents[i + 1] = "-1";
								break;
							}
						}
						for (int i = 0; i < mutantHostages.size(); i++) {
							String[] mHos = mutantHostages.get(i).split(",");
							if (kx == Integer.parseInt(mHos[0]) && ky == Integer.parseInt(mHos[1])) {
								// remove the mutant hostages from the arraylist
								mutantHostages.remove(i);
								break;
							}
						}
					}
				}
			} else {
				ArrayList<String> toKill = getPossibleKills(prevNode);
				neoD += 20;
				for (int w = 0; w < toKill.size(); w++) {
					// perform the kill
					// increment the number of killings by 1
					kills++;
					// increment Neo's damage by 20

					int kx = -1;
					int ky = -1;
					if (toKill.get(w).contains("KillUp")) {
						kx = prevNode.myLoc.x - 1;
						ky = prevNode.myLoc.y;
					} else if (toKill.get(w).contains("KillRight")) {
						kx = prevNode.myLoc.x;
						ky = prevNode.myLoc.y + 1;
					} else if (toKill.get(w).contains("KillDown")) {
						kx = prevNode.myLoc.x + 1;
						ky = prevNode.myLoc.y;
					} else if (toKill.get(w).contains("KillLeft")) {
						kx = prevNode.myLoc.x;
						ky = prevNode.myLoc.y - 1;
					}
					// update grid by removing the killed agent or mutant hostage
					for (int i = 0; i < agents.length - 1; i += 2) {
						if (kx == Integer.parseInt(agents[i]) && ky == Integer.parseInt(agents[i + 1])) {
							// replace the x and y with negatives so that when we combine the string again
							// we can know that this is not valid
							agents[i] = "-1";
							agents[i + 1] = "-1";
							break;
						}
					}
					for (int i = 0; i < mutantHostages.size(); i++) {
						String[] mHos = mutantHostages.get(i).split(",");
						if (kx == Integer.parseInt(mHos[0]) && ky == Integer.parseInt(mHos[1])) {
							// remove the mutant hostages from the arraylist
							mutantHostages.remove(i);
							break;
						}
					}
				}
			}
			break;
		case ("takePill"):
			// remove the pill
			for (int i = 0; i < pills.length - 1; i += 2) {
				if (prevNode.myLoc.x == Integer.parseInt(pills[i])
						&& prevNode.myLoc.y == Integer.parseInt(pills[i + 1])) {
					// replace the x and y with negatives so that when we combine the string again
					// we can know that this is not valid
					pills[i] = "-1";
					pills[i + 1] = "-1";
					break;
				}
			}
			// Decrement the damages of hostages and Neo by 20
			if (neoD < 20) {
				neoD = 0;
			} else {
				neoD -= 20;
			}
			for (int i = 0; i < hostages.size(); i++) {
				// each entry in the arraylist is a string with commas splitting the x and y and
				// damage
				String[] splitted2 = hostages.get(i).split(",");
				// the third element is the damage
				int oldDamage2 = Integer.parseInt(splitted2[2]);
				// check that is cannot reach below 0
				if (oldDamage2 < 20) {
					oldDamage2 = 0;
				} else {
					oldDamage2 -= 20;
				}
				// set the new damage
				hostages.set(i, splitted2[0] + "," + splitted2[1] + "," + oldDamage2);
			}
			break;
		case ("carry"):
			int cAllowed = Integer.parseInt(splitted[1]);
			// check that there is capacity
			if (carried.size() + 1 <= cAllowed) {
				for (int i = 0; i < hostages.size(); i++) {
					String[] hos = hostages.get(i).split(",");
					if (prevNode.myLoc.x == Integer.parseInt(hos[0]) && prevNode.myLoc.y == Integer.parseInt(hos[1])) {
						hostages.remove(i);
						carried.add(Integer.parseInt(hos[2]));
						break;
					}
				}
			}
			break;
		case ("drop"):
			// Reset carried
			if (carried.size() > 0) {
				carried = new ArrayList<Integer>();
			}
			break;
		case ("down"):
			// nothing changes in the grid only the overall hostage damage increases by 2
			// and this is already handled above
			newLocation = moveTo;
			break;
		case ("up"):
			// nothing changes in the grid only the overall hostage damage increases by 2
			// and this is already handled above
			newLocation = moveTo;
			break;
		case ("right"):
			// nothing changes in the grid only the overall hostage damage increases by 2
			// and this is already handled above
			newLocation = moveTo;
			break;
		case ("left"):
			// nothing changes in the grid only the overall hostage damage increases by 2
			// and this is already handled above
			newLocation = moveTo;
			break;
		case ("fly"):
			// nothing changes in the grid only the overall hostage damage increases by 2
			// and this is already handled above
			newLocation = moveTo;
			break;
		default:
			break;
		}

		// update damage of all hostages
		for (int i = 0; i < hostages.size(); i++) {
			// each entry in the arraylist is a string with commas splitting the x and y and
			// damage
			String[] splittedHos = hostages.get(i).split(",");
			// the third element is the damage
			int oldDamage = Integer.parseInt(splittedHos[2]) + 2;
			if (action.contains("takePill")) {
				// reverse the action
				oldDamage -= 2;
			}
			// set the new damage
			hostages.set(i, splittedHos[0] + "," + splittedHos[1] + "," + oldDamage);
			// check if it reached 100
			if (oldDamage >= 100) {
				// remove this hostage from the hostages array and add it to mutant
				mutantHostages.add(hostages.get(i));
				// System.out.println(hostages.get(i));
				hostages.remove(i);
				i--;
				deaths++;
				// as we do not add a different category in the grid string to represent the
				// mutant hostages
				// we rely on having the damage of 100 or greater to reflect this change
			}
		}
		for (int i = 0; i < carried.size(); i++) {
			int d = carried.get(i);
			// System.out.println(d);
			if (action.contains("takePill")) {
				if (d < 100) {
					if (d < 20) {
						carried.set(i, 0);
					} else {
						carried.set(i, d - 20);
					}
				}
			} else if (d < 100 && !(action.contains("drop"))) {
				carried.set(i, d + 2);
				if (carried.get(i) >= 100) {
					// System.out.println(carried.get(i));
					deaths++;
				}
			}
		}
		// then we need to form the new grid

		for (int i = 0; i < agents.length - 1; i += 2) {
			if (agents[i] == "-1") {
				// killed
				continue;
			} else {
				result += agents[i] + "," + agents[i + 1];
				if (i < agents.length - 3) {
					result += ",";
				}
			}
		}
		result += ";";
		for (int i = 0; i < pills.length - 1; i += 2) {
			if (pills[i] == "-1") {
				// taken
				continue;
			} else {
				result += pills[i] + "," + pills[i + 1];
				if (i < pills.length - 3) {
					result += ",";
				}
			}
		}
		result += ";" + pads + ";";
		for (int i = 0; i < hostages.size(); i++) {
			String[] curHos = hostages.get(i).split(",");
			result += curHos[0] + "," + curHos[1] + "," + curHos[2];
			if (i < hostages.size()) {
				result += ",";
			}
		}
		// add the mutant hostages to the hostages category in the grid
		for (int i = 0; i < mutantHostages.size(); i++) {
			String[] curHos = mutantHostages.get(i).split(",");
			result += curHos[0] + "," + curHos[1] + "," + "100";
			if (i < mutantHostages.size() - 1) {
				result += ",";
			}
		}
		// creating the resultant node
		TreeNode resNode = new TreeNode(prevNode, prevNodes, newLocation, neoD, result, kills, deaths, actionDetails[0],
				prevNode.depth + 1, 0, 0, carried);
		return resNode;
	}


	
	public static double calculateH(TreeNode node, String possibleAction) {
		// there is an alive hostage, he won't die until neo reaches him
		// return distance from neo to nearest hostage to telephone
		double cost = 0;
		String[] splitted = node.grid.split(";");
		String[] actionDetails = new String[3];
		Location affected = null;
		Location currentNeo = node.myLoc;
		int killMutantWeight = 60;
		String[] padsS = splitted[6].split(",");
		ArrayList<Location> padsLoc = new ArrayList<Location>();
		for (int i = 0; i < padsS.length - 1; i += 4) {
			Location startPadLoc = new Location(Integer.parseInt(padsS[i]), Integer.parseInt(padsS[i + 1]));
			Location finishPadLoc = new Location(Integer.parseInt(padsS[i + 2]), Integer.parseInt(padsS[i + 3]));
			padsLoc.add(startPadLoc);
			padsLoc.add(finishPadLoc);
		}
		// get distance to hostage
		// double distanceToClosestHostage = Integer.MAX_VALUE;
		double distanceToBooth = Integer.MAX_VALUE;
		boolean foundHostage = false;
		ArrayList<String> hostages = getHostages(node.grid);
		Location closHos = null;
		Location minHos = null;
		double maxDamage = 0;
		Location maxHos = null;
		double distanceToMax = Integer.MAX_VALUE;
		for (int i = 0; i < hostages.size(); i++) {
			String hostage = hostages.get(i);
			String[] hostageDetails = hostage.split(",");
			int hostageDamage = Integer.parseInt(hostageDetails[2]);
			closHos = new Location(Integer.parseInt(hostageDetails[0]), Integer.parseInt(hostageDetails[1]));
			double distance = calculatePyDistance2(closHos, currentNeo);
			double hostageDamageAfter = hostageDamage + (2 * distance);
			// check if pads are shorter
			for (int j = 0; j < padsLoc.size() - 1; j += 2) {
				Location startPadLoc = padsLoc.get(j);
				Location finishPadLoc = padsLoc.get(j + 1);
				double distanceToStart = calculatePyDistance2(startPadLoc, currentNeo);
				double distanceFromFinishToHostage = calculatePyDistance2(finishPadLoc, closHos);
				double padDistance = distanceToStart + distanceFromFinishToHostage;
				if (distance > padDistance) {
					distanceToMax = padDistance;
					hostageDamageAfter = hostageDamage + (2 * distanceToMax);
				} else {
					distanceToMax = distance;
				}
			}

			if (hostageDamageAfter < 100 && hostageDamage > maxDamage) {
				// make sure he won't be dead by the time neo reaches him
				maxDamage = hostageDamage;
				foundHostage = true;
				maxHos = closHos;
			}

			if (hostageDamageAfter >= 100) {
				continue;
			}

		}
		if (foundHostage) {
			// get distance to booth
			Location tele = getClosestLoc(maxHos, "telephone", node);
			// distanceToBooth = calculatePyDistance(tele, closHos);
			distanceToBooth = calculatePyDistance2(tele, maxHos);

			for (int j = 0; j < padsLoc.size() - 1; j += 2) {
				Location startPadLoc = padsLoc.get(j);
				Location finishPadLoc = padsLoc.get(j + 1);
				double distanceToStart = calculatePyDistance2(startPadLoc, maxHos);
				double distanceFromFinishToBooth = calculatePyDistance2(finishPadLoc, tele);
				double padDistance = distanceToStart + distanceFromFinishToBooth;
				if (distanceToBooth > padDistance) {
					distanceToBooth = padDistance;
				}
			}

			cost += (2 * (distanceToBooth + distanceToMax));
			// return cost;

		}
		// there are no alive hostages or they will die before neo reaches them
		// return distance from neo to nearest mutant + kill cost
		// get distance to mutant
		Location closMut = getClosestLoc(currentNeo, "mutant", node);
		double distanceToClosestMutant = 0;
		if (closMut != null) {
			distanceToClosestMutant = calculatePyDistance2(closMut, currentNeo);
			for (int j = 0; j < padsLoc.size() - 1; j += 2) {
				Location startPadLoc = padsLoc.get(j);
				Location finishPadLoc = padsLoc.get(j + 1);
				double distanceToStart = calculatePyDistance2(startPadLoc, currentNeo);
				double distanceFromFinishToMutant = calculatePyDistance2(finishPadLoc, closMut);
				double padDistance = distanceToStart + distanceFromFinishToMutant;
				if (distanceToClosestMutant > padDistance) {
					distanceToClosestMutant = padDistance;
				}
			}
			cost += (distanceToClosestMutant * killMutantWeight * node.neoDamage);
			// return cost;
		}

		// no hostages or mutants
		// return distance from neo to telephone
		Location tele = getClosestLoc(currentNeo, "telephone", node);
		distanceToBooth = calculatePyDistance2(tele, currentNeo);
		for (int j = 0; j < padsLoc.size() - 1; j += 2) {
			Location startPadLoc = padsLoc.get(j);
			Location finishPadLoc = padsLoc.get(j + 1);
			double distanceToStart = calculatePyDistance2(startPadLoc, currentNeo);
			double distanceFromFinishToBooth = calculatePyDistance2(finishPadLoc, tele);
			double padDistance = distanceToStart + distanceFromFinishToBooth;
			if (distanceToBooth > padDistance) {
				distanceToBooth = padDistance;
			}
		}
		cost += distanceToBooth;
		if (possibleAction.contains("kill")) {
			int kills = getPossibleKills(node).size();
			cost += kills * killMutantWeight;
		}
		// return cost;
		return cost;

	}

	public static double calculatePyDistance(Location loc1, Location loc2) {
		double result;
		result = Math.abs(loc1.x - loc2.x) + Math.abs(loc1.y - loc2.y);
		return result;
	}
	
	//pythagoras
	public static double calculatePyDistance2(Location loc1, Location loc2) {
		double result;
		result = Math.sqrt(((loc1.x - loc2.x) * (loc1.x - loc2.x)) + ((loc1.y - loc2.y) * (loc1.y - loc2.y)));
		return result;
	}

	public static Location getClosestLoc(Location loc, String obj, TreeNode node) {
		Location result = null;
		double min = Integer.MAX_VALUE;
		switch (obj) {
		case ("hostage"):
			ArrayList<String> hostages = getHostages(node.grid);
			for (int i = 0; i < hostages.size(); i++) {
				String[] hos = hostages.get(i).split(",");
				int hosx = Integer.parseInt(hos[0]);
				int hosy = Integer.parseInt(hos[1]);
				double distance = ((loc.x - hosx) * (loc.x - hosx)) + ((loc.y - hosy) * (loc.y - hosy));
				if (distance < min) {
					min = distance;
					result = new Location(hosx, hosy);
				}
			}
			return result;
		case ("mutant"):
			ArrayList<String> mutant = getMutantHostages(node.grid);
			for (int i = 0; i < mutant.size(); i++) {
				String[] hos = mutant.get(i).split(",");
				int hosx = Integer.parseInt(hos[0]);
				int hosy = Integer.parseInt(hos[1]);
				double distance = ((loc.x - hosx) * (loc.x - hosx)) + ((loc.y - hosy) * (loc.y - hosy));
				if (distance < min) {
					min = distance;
					result = new Location(hosx, hosy);
				}
			}
			return result;
		case ("pill"):
			String[] splitted = node.grid.split(";");
			// array that contains locations of all the hostages
			String[] pills;
			ArrayList<String> pillsToBeUsed = new ArrayList<String>();
			// if (splitted.length <= 7) {
			// pills = new String[0];
			// } else {
			pills = splitted[5].split(",");
			// }

			for (int i = 0; i < pills.length - 1; i += 2) {
				// store the location in a string
				String temp = "";
				temp += pills[i] + "," + pills[i + 1];
				pillsToBeUsed.add(temp);
			}

			for (int i = 0; i < pillsToBeUsed.size(); i++) {
				String[] pill = pillsToBeUsed.get(i).split(",");
				int pillx = Integer.parseInt(pill[0]);
				int pilly = Integer.parseInt(pill[1]);
				double distance = ((loc.x - pillx) * (loc.x - pillx)) + ((loc.y - pilly) * (loc.y - pilly));
				if (distance < min) {
					min = distance;
					result = new Location(pillx, pilly);
				}
			}
			return result;
		case ("pad"):
			splitted = node.grid.split(";");
			// array that contains locations of all the pads
			String[] pads;
			ArrayList<String> padsToBeUsed = new ArrayList<String>();
			pads = splitted[6].split(",");
			// if (splitted.length <= 7) {
			// pads = new String[0];
			// } else {
			// pads = splitted[6].split(",");
			// }

			for (int i = 0; i < pads.length - 1; i += 2) {
				// store the location in a string
				String temp = "";
				temp += pads[i] + "," + pads[i + 1];
				padsToBeUsed.add(temp);
			}

			for (int i = 0; i < padsToBeUsed.size(); i++) {
				String[] pad = padsToBeUsed.get(i).split(",");
				int padx = Integer.parseInt(pad[0]);
				int pady = Integer.parseInt(pad[1]);
				double distance = ((loc.x - padx) * (loc.x - padx)) + ((loc.y - pady) * (loc.y - pady));
				if (distance < min) {
					min = distance;
					result = new Location(padx, pady);
				}
			}
			return result;
		case ("telephone"):
			splitted = node.grid.split(";");
			// array that consists of (x,y) of the telephone booth
			String[] telephone = splitted[3].split(",");
			int telex = Integer.parseInt(telephone[0]);
			int teley = Integer.parseInt(telephone[1]);
			double distance = ((loc.x - telex) * (loc.x - telex)) + ((loc.y - teley) * (loc.y - teley));
			if (distance < min) {
				min = distance;
				result = new Location(telex, teley);
			}

		}
		return result;

	}

	// TODO check if correct
	public static double calculatehCost2(TreeNode node, String action) {
		double cost = 0;
		int deathWeight = 60;
		// int killAgentWeight = 8;
		int killMutantWeight = 50;
		int deaths = 0;
		ArrayList<Integer> carried = node.carried;
		ArrayList<String> hostages = getHostages(node.grid);
		int killNumMutant = 0;
		// int killNumAg = 0;
		// get the damages of the current hostages whether in grid or carried
		ArrayList<String> gridHostages = getHostages(node.grid);
		// if the action is not taking a pill then the damages will increase by 2
		if (!(action.contains("takePill"))) {
			for (int i = 0; i < gridHostages.size(); i++) {
				String[] splittedHos = gridHostages.get(i).split(",");
				// the third element is the damage
				int oldDamage = Integer.parseInt(splittedHos[2]) + 2;
				// check if it reached 100
				if (oldDamage >= 100) {
					deaths++;
				}
			}
			// check that the action is not drop
			if (!(action.contains("drop"))) {
				for (int i = 0; i < carried.size(); i++) {
					int d = carried.get(i);
					if (d < 100) {
						d += 2;
						if (d >= 100) {
							deaths++;
						}
					}
				}
			}
		}


		cost = (deaths * deathWeight) + (hostages.size() * 2) +node.depth + 1;
		// cost = (deaths * deathWeight) + (killNumAg * killAgentWeight) +
		// (killNumMutant * killMutantWeight)
		// + (2 * (gridHostages.size() + carried.size()));

		return cost;
	}

	// TODO check if correct
	public static double calculateActualCost(TreeNode node, String action) {
		double cost = 0;
		int deathWeight = 60;
		int killWeight = 60;
		// TODO check if needed to accumulate
		int deaths = node.deaths;
		ArrayList<Integer> carried = node.carried;
		int carriedNum = carried.size();
		int killNum = node.kills;
		// get the damages of the current hostages whether in grid or carried
		ArrayList<String> gridHostages = getHostages(node.grid);
		// if the action is not taking a pill then the damages will increase by 2
		// if (!(action.contains("takePill"))) {
		for (int i = 0; i < gridHostages.size(); i++) {
			String[] splittedHos = gridHostages.get(i).split(",");
			// the third element is the damage
			int oldDamage = Integer.parseInt(splittedHos[2]) + 2;
			// check if it reached 100
			if (oldDamage >= 100) {
				deaths++;
			}
		}
		// check that the action is not drop
		if (!(action.contains("drop"))) {
			for (int i = 0; i < carried.size(); i++) {
				int d = carried.get(i);
				if (d < 100) {
					d += 2;
					if (d >= 100) {
						deaths++;
						carriedNum--;
					}
				} else {
					carriedNum--;
				}
			}
		}
		if (action.contains("kill")) {
			ArrayList<String> kills = getPossibleKills(node);
			killNum += kills.size();

		}
		cost = (deaths * deathWeight) + (killNum * killWeight) + node.depth + 1;
		// + (droppedAlive * rescuedWeightAlive) + (droppedDead * rescuedWeightDead)+
		// pillWeight

		return cost;
	}


}


