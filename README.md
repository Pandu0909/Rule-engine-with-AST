import ast  # Importing Abstract Syntax Tree module
import operator  # Importing the operator module for operators

class SafeEvaluator(ast.NodeVisitor):#create class and inherit the ast
    # Mapping of supported operators to their corresponding functions
    operators = {
        ast.Eq: operator.eq,    # Equality     (==)
        ast.NotEq: operator.ne, # Not equal    (!=)
        ast.Lt: operator.lt,    # Less than     (<)
        ast.LtE: operator.le,   # Less than or equal (<=)
        ast.Gt: operator.gt,    # Greater than  (>)
        ast.GtE: operator.ge,   # Greater than or equal (>=)
        ast.And: operator.and_, # Logical (AND)
        ast.Or: operator.or_,   # Logical (OR)
    }

    def __init__(self, data):
        self.data = data  # Initialize with the data dictionary for variable replacement

    def visit_Name(self, node):
        # Visit a variable name and replace it with the actual value from the data dictionary
        if node.id in self.data:
            return self.data[node.id]  # Return the value corresponding to the variable
        raise ValueError(f"Unknown variable: {node.id}")  # Raise an error if the variable is not found

    def visit_Constant(self, node):
        # Visit a constant value (eg. numbers, strings) and return it as-is
        return node.value

    def visit_Compare(self, node):
        # Visit a comparison operation (e.g. age > 40)
        left = self.visit(node.left)  # Evaluate the left-hand side of the comparison
        for op, comparator in zip(node.ops, node.comparators):
            right = self.visit(comparator)  # Evaluate the right-hand side
            # Use the operator mapping to perform the comparison
            if not self.operators[type(op)](left, right):
                return False  # If any comparison fails, return False
        return True  # All comparisons succeeded

    def visit_BoolOp(self, node):
        # Visit a boolean operation (e.g., and/or)
        if isinstance(node.op, ast.And):
            # Logical AND: return True if all sub-expressions are True
            return all(self.visit(value) for value in node.values)
        elif isinstance(node.op, ast.Or):
            # Logical OR: return True if any sub-expression is True
            return any(self.visit(value) for value in node.values)
        else:
            raise ValueError("Unsupported boolean operation")  # Error if an unsupported operation is used

    def evaluate(self, expression):
        # Parse the expression into an AST and evaluate it
        tree = ast.parse(expression, mode='eval')  # Parse the expression string into an AST
        return self.visit(tree.body)  # Start evaluation from the body of the parsed AST

class RuleEngine:
    def evaluate(self, rule, data):
        # Create an instance of SafeEvaluator and evaluate the rule against the provided data
        evaluator = SafeEvaluator(data)
        return evaluator.evaluate(rule)

if __name__ == "__main__":
    engine = RuleEngine()  # Create an instance of the RuleEngine
    
    # Define rules as strings
    rule_1 = "age > 40 and department == 'HR'"
    rule_2 = "salary > 50000 or experience > 5"
    
    # Combine rules using logical operators
    combined_rule = f"({rule_1}) or ({rule_2})"
    
    # Sample data to be evaluated against the rules
    data = {
        "age": 35,
        "department": "Sales",
        "salary": 60000,
        "experience": 6
    }
    
    # Evaluate the combined rule using the RuleEngine
    result = engine.evaluate(combined_rule, data)
    print(f"Evaluation result: {result}")  # Output the result of the combined rule evaluation

    # Modify a rule and evaluate again
    new_rule = "age > 30 and department == 'Sales'"
    modified_result = engine.evaluate(new_rule, data)
    print(f"Modified rule evaluation result: {modified_result}")  # Output the result of the modified rule evaluation
