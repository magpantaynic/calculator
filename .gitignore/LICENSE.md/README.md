# calculator
// Just a major project from my first coding class

#include "std_lib_facilities.h"

//------------------------------------------------------------

class Token {
	public:
		char kind;        // what kind of token
		double value;     // for numbers: a value

		Token()	// make a trivial token
		{
			kind = 0;
			value = 0;
		}
		Token(char ch)    // make a Token from a char
		{
			kind = ch; 
			value = 0; 
		}    
		Token(char ch, double val)     // make a Token from a char and a double
		{
			kind = ch; 
			value = val; 
		}
		friend ostream& operator<< (ostream& out, const Token& tok )  // in order for "cout << token;" to work
		{
			if ( tok.kind == '8' )
				out << tok.value;
			else 
				out << tok.kind;
			return out;
		}
};

//------------------------------------------------------------

// prints the tokens of the vector (it should look like an algebraic expression)
void print( const vector<Token>& v )
{
	// write your code here
	int i = 0;
	if (v.empty()) {
			cout << "{ }" ;
	}
	for(i=0; i < v.size(); i++)
		cout << v[i];

}

//------------------------------------------------------------

Token get_token()    // read a token from cin
{
	Token tok;
	char ch;
	cin >> ch;    // note that >> skips whitespace (space, newline, tab, etc.)

	switch (ch) {
		case '(': case ')': case '+': case '-': case '*': case '/': case ';': case 'q': 
			{
				tok = Token(ch);        // let each character represent itself
				break;
			}
		case '.':
		case '0': case '1': case '2': case '3': case '4':
		case '5': case '6': case '7': case '8': case '9':
			{    
				cin.putback(ch);         // put digit back into the input stream
				double val;
				cin >> val;              // read a floating-point number
				tok = Token('8',val);   // let '8' represent "a number"
				break;
			}
		default:
			error("Bad token");
	}

	return tok;
}

//------------------------------------------------------------

// remove len tokens from vector starting with v[pos]
void remove_token( vector<Token>& v, size_t pos, size_t len ) {
	// starting pos is too large
	if ( pos >= v.size() )   
		return;

	// starting pos is ok, remove from v[pos] on 
	if ( pos+len >= v.size() )   
	{
		if ( pos == 0 )
			v.clear();   // clear the whole vector
		else 
		{
			size_t j = v.size() - pos; // num of elem to remove
			for (size_t i = 0; i < j; i++ )
				v.pop_back();
		}
		return;
	}

	// the case when pos+len < v.size()
	size_t i,j;
	// copy the tail of the vector over starting with v[pos]
	for (i = pos+len, j = pos; i < v.size(); i++, j++ )
		v[j] = v[i];
	// remove the last len entries
	for (i = 0; i < len; i++ )
		v.pop_back();	

}

//------------------------------------------------------------

// to evaluate a subexpression between v[beg] and v[end] 
// not containing parantheses; tokens are removed in the process
double evaluate_subexpression( vector<Token>&v, size_t beg, size_t end )
{
		if ( beg == end )
			return v[beg].value;
		
		double subval;
		// write your code here
		
		int i;
		subval = 0.0;
		
		i = beg+1;
		while (i < end)
		{
				if (v[i].kind == '*')
				{
						v[i-1].value *= v[i+1].value;
						remove_token (v,i,2);
						end -= 2;
				}
				else if (v[i].kind == '/')
				{
						v[i-1].value /= v[i+1].value;
						remove_token (v,i,2);
						end -= 2;
				}
				else
				{
						i++;
				}
		}
		
		subval = v[beg].value;
		for (i = beg+1; i < end; i++)
		{
					if (v[i].kind == '+')
					{
							subval += v[i+1].value;
					}
					else if (v[i].kind == '-')
					{
						subval -= v[i+1].value;
					}
		}
		v[beg].value = subval;
		remove_token (v, beg+1, end-beg);
		cout << "subval = " << subval << endl;
		return subval;
					
}

//------------------------------------------------------------
// to evaluate an expression possibly containing parantheses;
// it calls evaluate_subexpression;
// it is responsible for removing parantheses once the expression between () is computed
double evaluate( vector<Token>& v ) 
{
	double val = 0.0;
	size_t beg, end;   	// position of rightmost '(' and left most ')' 
	bool lparenfound, rparenfound;
	int i;
	
	if ( v.size() == 1 )
	{   
		val = v[0].value;
		v.clear();
		return val;
	}

	// write your code here 
	while ( v.size() > 1 )
	{
		beg = 0;
		end = v.size()-1;
		
		lparenfound = rparenfound = false;
		for (i = end; i > 0; i--)
		{
			if (v[i].kind == ')')
			{
				end = i;
				lparenfound = true;
			}
		}
		for (i = 0; i < end; i++)
		{
			if (v[i].kind == '(')
			{
				beg = i;
				rparenfound = true;
			}
		}
		
		if (lparenfound != rparenfound)
		{
			cout << "error";
			return 0.0;
		}
		if (lparenfound)
		{
			cout << beg+1 << " " << end-1 << endl;
			val = evaluate_subexpression(v, beg+1, end-1);
			remove_token (v, beg, 1);
			remove_token (v, beg+1, 1);
		}
		else
		{
			val = evaluate_subexpression (v, beg, end);
		}
	}
		
		v.clear();
		return val;
}


//------------------------------------------------------------
int main()
{
	try
	{
		vector<Token> v;
		Token tok;
		double val = 0;

		cout << "Welcome to our calculator! Enter an expression.\n" 
			<< "End with ';' to evaluate.\n"
			<< "End with 'q' to quit.\n";
		while ( true ) 
		{		
			do {
				tok = get_token();
				if ( tok.kind != ';' and tok.kind != 'q' )
					v.push_back(tok);

			} while ( tok.kind != ';' and tok.kind != 'q' );
			
			if ( tok.kind == ';'  ) 
			{	
				val = evaluate(v);
				cout <<  val;
				cout << "\n----------------------------------\n";
			}
			else // quit 
			{
				if ( ! v.empty() ) // eval expressions such as 1+2*3q
				{
					val = evaluate(v);
					cout <<  val;
					cout << "\n----------------------------------\n";
				}
				cout << "Calculator exits.\n";
				break;
			}	

		}
	}
	catch (exception& e) {
		cerr << "error: " << e.what() << '\n'; 
		return 1;
	}
	catch (...) {
		cerr << "Oops: unknown exception!\n"; 
		return 2;
	}
	return 0;
}
//------------------------------------------------------------
