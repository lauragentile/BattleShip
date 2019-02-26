# BattleShip
Play a game of Battleship against a computer

package battleship;

import java.util.Random;

public class Ocean {
	
	/**
	 * Used	to quickly determine which ship is in any given location
	 */
	private Ship[][] ships = new Ship[10][10];
	
	/**
	 * The total number of shots fired by the user
	 */
	private int shotsFired;

	/**
	 * The number of times a shot hit a ship. If the user shoots the same part of	
	 * a ship more than once, every hit is counted, even though additional “hits”	
	 * don’t do the user any good.
	 */
	private int hitCount;	
	
	/**
	 * The number of ships sunk (10 ships in all)
	 */
	private int shipsSunk;
	
	/**
	 * Creates an ”empty” ocean and fills the ships array with EmptySea objects.
	 * Also initializes game variables, such as how many shots have been fired and how many ships have been sunk, and how many hits have been made.
	 */
	public Ocean() {
		
		//Initializes game variables 
		shotsFired = 0;
		hitCount = 0;
		shipsSunk = 0;
		
		//for each cell in the ships array, set the cell to an EmptySea ship and set the corresponding 
		//bow row, column, and horizontal value of each EmptySea ship
		for (int i = 0; i < ships.length; i++) {
			for (int j = 0; j < ships[i].length; j++){
				ships[i][j] = new EmptySea();
				ships[i][j].setBowRow(i);
				ships[i][j].setBowColumn(j);
				ships[i][j].setHorizontal(true);
			}
		}
	}

	/**
	 * Places all ten ships randomly on the ocean. This includes generating a random bow row number, bow column number, and horizontal value for
	 * each ship and checking to see if the ship can be legally placed in that location. 
	 * 
	 */
	void placeAllShipsRandomly() {
		
		//Initializing random number generator 
		Random rand = new Random();
		
		//creating an array of all the ships that need to be placed (10 in all)
		Ship[] shipsToPlace = {new Battleship(), new Cruiser(), new Cruiser(), new Destroyer(), new Destroyer(), 
				new Destroyer(), new Submarine(), new Submarine(), new Submarine(), new Submarine()};
		
		//For each ship in the array, creates a while loop that will continually generate a random bow row, bow column, 
		//and horizontal value for each ship until its finds a legal place to put each ship
		for (Ship n : shipsToPlace) {
			boolean l = false;
			while (l == false) {
				int r = rand.nextInt(10);
				int c = rand.nextInt(10);
				boolean b = rand.nextBoolean();
				
				//checks to see if ship can be placed at the location specified by the random variables
				if (n.okToPlaceShipAt(r, c, b, this)){
					
					//if okay to place ship at is true, places ship within the ocean and ends the while loop for that ship
					n.placeShipAt(r, c, b, this);
					l = true;
				}
			}
		}
	}

	/**
	 * Returns true if the given location contains a ship, false if it does not
	 * 
	 * @param row to check
	 * @param column to check
	 * @return boolean indicating if location is occupied
	 */
	boolean isOccupied(int row, int column) {
		
		//checks to see if the ship at the given location is an EmptySea ship. If so, location is not occupied. Return false.
		if ("empty".equals(this.getShipArray()[row][column].getShipType())) {
			return false;
			
		//otherwise, return true
		} else {
			return true;
		}
	}

	/**
	 * Returns true if the given location contains a ”real” ship, still afloat, (not
	 * an EmptySea), false if it does not. In addition, this method updates the
	 * number of shots that have been fired, the number of hits, and the number of sunk ships. Note: If a
	 * location contains a “real” ship, shootAt should return true every time the
	 * user shoots at that same location. Once a ship has been ”sunk”, additional
	 * shots at its location should return false.
	 * 
	 * @param row to shoot at
	 * @param column to shoot at
	 * @return boolean indicating if a 'real' un-sunk ship was hit (i.e. not EmptySea ship)
	 */
	boolean shootAt(int row, int column) {
		
		//increases the number of shots fired by one
		shotsFired++;
		
		//if a real ship is located at the given coordinates, increases the hitCount
		if (ships[row][column].shootAt(row, column)) {
			hitCount++;
			
			//if the ship hit is now sunk, increases the ship sunk count by one
			if (ships[row][column].isSunk()) {
				shipsSunk++;
			}
			
			//returns true since a real ship was hit
			return true;
			
		//if location does not contain a real ship, return false	
		} else {
			return false;
		}	
	}
	
	/**
	 * Returns true if all ships have been sunk, otherwise false
	 * 
	 * @return boolean indicating if game is over 
	 */
	boolean isGameOver() {
		
		//if the total count of ships sunk is 10, return true
		if (shipsSunk == 10) { 
			return true;
			
		//otherwise return false	
		} else{
			return false;
		}
	}

	/**
	 * Returns true if the row/column has previously been fired at. Otherwise, returns false. 
	 * This method is used within the print method in the ocean class.
	 * @param row to check
	 * @param column to check
	 * @return boolean indicating if location has been shot at
	 */
	private boolean shotAt(int row, int column) {
		
		//creating a new variable to reference the ship located at the given row and column number
		Ship ship = ships[row][column];
		
		//if the ship is horizontally placed...
		if (ship.isHorizontal()) {
			
			//checks to see if the location of that ship contains a true value in its hit array. Does this by finding out how 
			//far the given column location is from the ships bow column and then checking that same location in the ship's 
			//hit array to see if it contains a 'true' boolean. If so, returns true to indicate that that location has been shot at before
			int distance = column - ship.getBowColumn();
			if (ship.getHit()[distance]) {
				return true;
			}
		
		//if the ship is vertically placed...
		} else {
			
			//checks to see if the location of that ship contains a true value in its hit array. Does this by finding out how 
			//far the given row location is from the ships bow row and then checking that same location in the ship's 
			//hit array to see if it contains a 'true' boolean. If so, returns true to indicate that that location has been shot at before
			int distance2 = row - ship.getBowRow();
			if (ship.getHit()[distance2]) {
				return true;
			}
		}
		
		//if the boolean in the ships hit array does not equal true (i.e. that location has not been hit before) then returns false
		return false;
	}
		
	/**
	 * Prints the ocean with row numbers displayed along the left edge of the array, and column numbers displayed along the top. 
	 * Uses ‘S’ to indicate a location that has been fired upon and hit a real ship
	 * Uses ‘-’ to indicate a location that you have fired upon and found only an EmptySea ship (i.e. not a real ship)
	 * Uses ‘x’ to indicate a location containing a sunken real ship 
	 * Uses ‘.’ to indicate a location that has never been fired upon
	 * 
	 */
	void print() {
		
		//prints column numbers
		System.out.print('\n');
		System.out.print("  0 1 2 3 4 5 6 7 8 9");
		
		//starting at row 0..
		int row = 0;
		
		//for each row from 0 to 9, print the row number...
		for (int i = 0; i < 10; i++) {
			System.out.print('\n');
			System.out.print(row);
			//increase row number by one
			row++;
			
			//for each column number within that row, check to see if the location has been shot at. 
			for (int j = 0; j < 10; j++) {
				if (this.shotAt(i, j)) {
					
					//if so, calls the toString method of each ship to print the correct indicator in the ocean
					System.out.print(ships[i][j]);
				
				//if location has not been shot at, prints a '.'
				} else {
					System.out.print(" .");
				}
			}
		}
	}
	
	//Getter and Setters
	
	/**
	 * Returns the number of shots fired (in the game)
	 * 
	 * @return int indicating the number of shots fired
	 */
	int getShotsFired() {
		return shotsFired;
	}

	/**
	 * Returns the number of hits recorded (in the game). All hits are counted, not
	 * just the first time a given square is hit.
	 * 
	 * @return int indicating the number of hits
	 */
	int getHitCount() {
		return hitCount;
	}

	/**
	 * Returns the number of ships sunk (in the game)
	 * 
	 * @return int indicating the number of ships sunk
	 */
	int getShipsSunk() {
		return shipsSunk;
	}

	/**
	 * Returns the 10x10 array of Ships.  
	 * 
	 * @return 10x10 ship array 
	 */
	Ship[][] getShipArray() {
		return ships;
	}
}
